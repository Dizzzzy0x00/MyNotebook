---
description: Glibc堆利用之house of系列
---

# House of

{% embed url="https://www.bilibili.com/video/BV1Js4y1N7kr" %}
学习参考教程
{% endembed %}

{% embed url="https://github.com/shellphish/how2heap" %}

## 前言

在house of 系列学习之前先阅读glibc（2.23）的堆块管理部分的源码

### malloc部分源码分析 <a href="#articlecontentid" id="articlecontentid"></a>

第一次调用malloc时会从\_\_malloc\_hook中取出malloc\_hook\_ini函数指针并执行

```
static void *
malloc_hook_ini (size_t sz, const void *caller)
{
	__malloc_hook = NULL;			//将__malloc_hook置0
	ptmalloc_init ();				//初始化ptmalloc
	return __libc_malloc (sz);		//回到__libc_malloc
}
```

接下来是\_\_libc\_malloc部分：

```c
void *
__libc_malloc(size_t bytes)
{
  //首先检查是否存在内存分配的 hook 函数，如果存在，调用 hook 函数，并返回，hook 函数主要用于进程在创建新线程过程中分配内存，或者支持用户提供的内存分配函数。
  mstate ar_ptr;
  void *victim;
  
  //判断__malloc_hook中是否有值，有值就当成函数指针调用
  void *(*hook)(size_t, const void *) = atomic_forced_read(__malloc_hook);
  //atomic_forced_read 是汇编语句，用于原子读操作，
  //每次只会读取一次，例如调用 malloc_hook_ini 初始化，只会调用一次
  if (__builtin_expect(hook != NULL, 0))
    return (*hook)(bytes, RETURN_ADDRESS(0));

  //获取分配区指针，如果获取分配区失败，返回退出，否则，调用 _int_malloc() 函数分配内存。
  arena_get(ar_ptr, bytes);

  victim = _int_malloc(ar_ptr, bytes); //分配内存在函数_int_malloc中实现
  /* Retry with another arena only if we were able to find a usable arena
     before.  */
    
  
  //如果 _int_malloc() 函数分配内存失败，就会判断使用的分配区是不是主分配区，然后是一些获取分配区，解锁之类的操作。
  if (!victim && ar_ptr != NULL)
  {
    LIBC_PROBE(memory_malloc_retry, 1, bytes);
    ar_ptr = arena_get_retry(ar_ptr, bytes);
    victim = _int_malloc(ar_ptr, bytes);
  }

  if (ar_ptr != NULL)
    (void)mutex_unlock(&ar_ptr->mutex);

  assert(!victim || chunk_is_mmapped(mem2chunk(victim)) ||
         ar_ptr == arena_for_chunk(mem2chunk(victim)));
  return victim;
}

```

总结一下运行流程：

第一次调用 malloc 申请堆空间：首先会跟着 hook 指针进入 `malloc_hook_ini()` 函数里面进行对 ptmalloc 的初始化工作，并置空 hook，再调用 `ptmalloc_init()` 和 `__libc_malloc()`；

再次调用 malloc 申请堆空间：malloc() -> \_\_libc\_malloc() -> \_int\_malloc()

然后是核心的函数 **`_int_malloc()`**

<pre class="language-c"><code class="lang-c">static void *
_int_malloc(mstate av, size_t bytes)
{
	  INTERNAL_SIZE_T nb; /* 符合要求的请求大小 */
	  unsigned int idx;   /* 相关的bin指数 */
	  mbinptr bin;        /* 相关的bin */
	
	  mchunkptr victim;     /* 检查/选择的块 */
	  INTERNAL_SIZE_T size; /* its size */
	  int victim_index;     /* its bin index */
	
	  mchunkptr remainder;          /* 被分割的剩余部分 */
	  unsigned long remainder_size; /* its size */
	
	  unsigned int block; /* bit map traverser */
	  unsigned int bit;   /* bit map traverser */
	  unsigned int map;   /* current word of binmap */
	
	  mchunkptr fwd; /* misc temp for linking */
	  mchunkptr bck; /* misc temp for linking */
	
	  const char *errstr = NULL;
	
	
	  checked_request2size(bytes, nb);//取得对齐后的size值赋值给nb
	  //checked_request2size将请求分配的内存大小 bytes 转换为需要分配下去的 chunk 实际大小 nb。
	  //Ptmalloc 内部分配都是以 chunk 为单位，根据 chunk 的大小
	  //决定如何分配满足条件的 chunk，达到分配最小的 size 的同时符合对齐要求的目的
	  
	  /* 没有可用的arena，随机分配一块内存并返回 */
	  if (__glibc_unlikely(av == NULL)){
	void* p = sysmalloc(nb, av);
	if (p != NULL)
	  alloc_perturb(p, bytes);
	        return p;
	  }
	  //先检查是否属于 fast bins 范围内，尝试分配链中的 chunk 出去
	  if ((unsigned long)(nb) &#x3C;= (unsigned long)(get_max_fast()))
	  {
	    //根据所需 chunk 的大小获得该 chunk 所属 fast bin 的 index。
	    idx = fastbin_index(nb);
	      
	    //从链中取出第一个 chunk，并调用 chunk2mem() 函数返回用户所需的内存块。
	    mfastbinptr *fb = &#x26;fastbin(av, idx);
	    mchunkptr pp = *fb;
	    do
	    {
	      victim = pp;
	      if (victim == NULL)
	        break;
	    } while ((pp = catomic_compare_and_exchange_val_acq(fb, victim->fd, victim)) != victim);
	    if (victim != 0)
	    {
	      if (__builtin_expect(fastbin_index(chunksize(victim)) != idx, 0))
	      {
	        errstr = "malloc(): memory corruption (fast)";
	      errout:
	        malloc_printerr(check_action, errstr, chunk2mem(victim), av);
	        return NULL;
	      }
	      check_remalloced_chunk(av, victim, nb);
	      void *p = chunk2mem(victim);
	      alloc_perturb(p, bytes);
	      return p;
	    }
	  }
	  //然后检查small bin
	  if (in_smallbin_range(nb))
	  {
	    idx = smallbin_index(nb);
	      
	    //根据 index 获得某个 small bin 的空闲 chunk 双向循环链表表头,在 if 语句里将最后一个 chunk 赋值给 victim。
	    bin = bin_at(av, idx);
	    //如果 victim 与表头相同，表示该链表为空，不能从 small bin 的空闲 chunk 链表中分配。
	    //下面都是 victim 与表头不相同的情况。
	    if ((victim = last(bin)) != bin)
	    {
	      //如果 victim 为 0，表示所属 small bin 还没有初始化为双向循环链表，调用 malloc_consolidate() 函数将 fast bins 中的 chunk 合并。
	      if (victim == 0) /* initialization check */
	        malloc_consolidate(av);
	      //否则说明有合适的 chunk 在对应的 bin 链，将 victim 从 small bin 的双向循环链表中取出，设置 victim chunk 的 inuse 标志，该标志处于 victim chunk 的下一个相邻 chunk 的 size 字段的第一个 bit。从 small bin 中取出 victim 也可以用 unlink() 宏函数，只是这里没有使用。
	      else
	      {
	        bck = victim->bk;
	        //经典的通过检查 victim 的 bck 的 fd 指针是否指向 victim，来确定链表是否有被破坏。
	        if (__glibc_unlikely(bck->fd != victim))
	        {
	          errstr = "malloc(): smallbin double linked list corrupted";
	          goto errout;
	        }
	        //脱链。
	        set_inuse_bit_at_offset(victim, nb);
	        bin->bk = bck;
	        bck->fd = bin;
	          
	        //接着判断当前分配区是否为非主分配区，如果是，将 victim chunk 的 size 字段中的表示非主分配区的标志 bit 清零，最后调用 chunk2mem() 函数获得 chunk 的实际可用的内存指针，将该内存指针返回给应用层。
	        if (av != &#x26;main_arena)
	          victim->size |= NON_MAIN_ARENA;
	        check_malloced_chunk(av, victim, nb);
	        void *p = chunk2mem(victim);
	        alloc_perturb(p, bytes);
	        return p;
	      }
	    }
	  }
//有两种情况当大小符合small bin但是无法获取chunk的情况：
// 1.当对应的 small bin 中没有空闲 chunk。
// 2.对应的 small bin 还没有初始化完成，这时就会调用 malloc_consolidate。

//如果 chunk 不属于 small bins，那么就一定属于 large bins，这里只直接进行 malloc_consolidate

<strong>	else {
</strong>		idx = largebin_index (nb);        //计算所需大小对应large bins的下标
		if (have_fastchunks (av))         //判断是否存在属于fast bins的空闲chunk
			malloc_consolidate (av);        //合并所有的fast bin
	}
	
	//可能会发生bin中chunk切割分配的操作，接下来的循环处理分割的chunk，unsorted bin
	for (;; )
	{
		int iters = 0;
		/* 反向遍历unsorted bins双向循环链表，直到候选chunk指向头节点 */
		while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av))
		{
			bck = victim->bk;
			//判断chunk大小是否合法
			if (__builtin_expect (victim->size &#x3C;= 2 * SIZE_SZ, 0)
				|| __builtin_expect (victim->size > av->system_mem, 0))
					/* 如果不合法就执行malloc_printerr打印错误信息 */
					malloc_printerr (check_action, "malloc(): memory corruption", chunk2mem (victim), av);
			
			size = chunksize (victim);     //若合法则取出size位
			
			/*
				If a small request, try to use last remainder if it is the
				only chunk in unsorted bin.  This helps promote locality for
				runs of consecutive small requests. This is the only
				exception to best-fit, and applies only when there is
				no exact fit for a small chunk.
			*/
			
			/* 如果这个chunk大小属于small bins
			   且unsorted bins中只有一个chunk，
			   且这个chunk为last remainder chunk，
			   且这个chunk的大小大于所需的size+MINSIZE */
			if (in_smallbin_range (nb) &#x26;&#x26;
				bck == unsorted_chunks (av) &#x26;&#x26;
				victim == av->last_remainder &#x26;&#x26;
				(unsigned long) (size) > (unsigned long) (nb + MINSIZE))
			{
				/* split and reattach remainder */
				/* 从这个remainder中取出所需的部分，与表头形成双向循环链表 */
				remainder_size = size - nb;       //计算取出所需部分后的剩余部分
				remainder = chunk_at_offset (victim, nb);    //获得chunk指针
				unsorted_chunks (av)->bk = unsorted_chunks (av)->fd = remainder;  //arena指向remainder
				av->last_remainder = remainder;       //设置新的remainder
				remainder->bk = remainder->fd = unsorted_chunks (av);             //remainder指向arena
				
				/* 如果剩余部分大小不属于small bins，则只能时largebins
				   因此需要将fd_nextsize和bk_nextsize清空，unsorted bin无需这两个成员 */
				if (!in_smallbin_range (remainder_size))
				{
					remainder->fd_nextsize = NULL;
					remainder->bk_nextsize = NULL;
				}
				
				/* 设置chunk的相关信息 */
				set_head (victim, nb | PREV_INUSE |
				(av != &#x26;main_arena ? NON_MAIN_ARENA : 0));
				set_head (remainder, remainder_size | PREV_INUSE);
				set_foot (remainder, remainder_size);
				
				check_malloced_chunk (av, victim, nb);
				void *p = chunk2mem (victim);     //取得用户部分可用的内存指针
				alloc_perturb (p, bytes);
				return p; //返回应用层
			}
			
			/* remove from unsorted list */
			/* 将bin从unsortedbin中取出 */
			unsorted_chunks (av)->bk = bck;
			bck->fd = unsorted_chunks (av);
			
			/* Take now instead of binning if exact fit */
			
			/* 若size位等于所需大小，则设置标志位，然后将bin取出并返回用户指针 */
			if (size == nb)
			{
				set_inuse_bit_at_offset (victim, size);
				if (av != &#x26;main_arena)
				victim->size |= NON_MAIN_ARENA;
				check_malloced_chunk (av, victim, nb);
				void *p = chunk2mem (victim);
				alloc_perturb (p, bytes);
				return p;     //返回应用层
			}
			
			/* place chunk in bin */
			/* 若size属于small bins，则将chunk加入到bck和fwd之间，作为small bins的第一个chunk */
			if (in_smallbin_range (size))
			{
				victim_index = smallbin_index (size);
				bck = bin_at (av, victim_index);
				fwd = bck->fd;
			}
			
			/* 若size属于large bins，则将chunk加入到bck和fwd之间，作为large bin的第一个chunk */
			else
			{
				victim_index = largebin_index (size);
				bck = bin_at (av, victim_index);
				fwd = bck->fd;
				
				/* maintain large bins in sorted order */
				if (fwd != bck)   //若fwd不等于bck，说明large bins中存在空闲chunk
				{
					/* Or with inuse bit to speed comparisons */
					size |= PREV_INUSE;
					/* if smaller than smallest, bypass loop below */
					assert ((bck->bk->size &#x26; NON_MAIN_ARENA) == 0);
					
					/* 如果当前size比最后一个chunk size还要小，则将当前size的chunk加入到chunk size链表尾
					   然后将所有大小的链表取出首个chunk链到一起，方便查找 */
					if ((unsigned long) (size) &#x3C; (unsigned long) (bck->bk->size))
					{
						fwd = bck;
						bck = bck->bk;
						
						victim->fd_nextsize = fwd->fd;
						victim->bk_nextsize = fwd->fd->bk_nextsize;
						fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
					}
					else
					{
						assert ((fwd->size &#x26; NON_MAIN_ARENA) == 0);
						/* 正向遍历chunk size链表，找到第一个chunk大小小于等于当前大小的chunk */
						while ((unsigned long) size &#x3C; fwd->size)
						{
							fwd = fwd->fd_nextsize;
							assert ((fwd->size &#x26; NON_MAIN_ARENA) == 0);
						}
						
						/* 若已经存在相同大小的chunk，则将当前chunk插入到同大小chunk链表的尾部 */
						if ((unsigned long) size == (unsigned long) fwd->size)
							/* Always insert in the second position.  */
							fwd = fwd->fd;
						/* 否则延伸出一个大小等于当前size的chunk链表，将该链表加入到chunk size链表尾 */
						else
						{
							victim->fd_nextsize = fwd;
							victim->bk_nextsize = fwd->bk_nextsize;
							fwd->bk_nextsize = victim;
							victim->bk_nextsize->fd_nextsize = victim;
						}
						bck = fwd->bk;
					}
				}
				else  //large bins中没有 chunk，直接将当前 chunk 加入 chunk size链表
					victim->fd_nextsize = victim->bk_nextsize = victim;
			}
			
			/* 将当前chunk加入large bins的空闲链表中 */
			mark_bin (av, victim_index);
			victim->bk = bck;
			victim->fd = fwd;
			fwd->bk = victim;
			bck->fd = victim;
			
			/* 最多遍历10000个unsorted bin，节约时间 */
			#define MAX_ITERS       10000
			if (++iters >= MAX_ITERS)
			break;
		}
		
		/*
			If a large request, scan through the chunks of current bin in
			sorted order to find smallest that fits.  Use the skip list for this.
		*/
		
		/* 当处理完unsorted bins后，使用最佳匹配法匹配chunk */
		
		if (!in_smallbin_range (nb))      //判断chunk是否位于large bins中
		{
			bin = bin_at (av, idx);
			
			/* skip scan if empty or largest chunk is too small */
			/* 判断large bins是否为空，以及链表中的最大size是否满足所需大小 */
			if ((victim = first (bin)) != bin &#x26;&#x26; (unsigned long) (victim->size) >= (unsigned long) (nb))
			{
				/* 遍历chunk size链表，找到大于等于所需大小的chunk链表 */
				victim = victim->bk_nextsize;
				while (((unsigned long) (size = chunksize (victim)) &#x3C; (unsigned long) (nb)))
					victim = victim->bk_nextsize;
				
				/* Avoid removing the first entry for a size so that the skip
				   list does not have to be rerouted.  */
				/* 为了尽量不破坏链表结构，尝试取出victim->fd作为候选chunk */
				if (victim != last (bin) &#x26;&#x26; victim->size == victim->fd->size)
				victim = victim->fd;
				
				/* 计算剩余size，然后断链 */
				remainder_size = size - nb;
				unlink (av, victim, bck, fwd);
				
				/* Exhaust */
				/* 若剩余部分小于MIN_SIZE，则将整个chunk分配给应用层（可以搞事情嗷） */
				if (remainder_size &#x3C; MINSIZE)
				{
					set_inuse_bit_at_offset (victim, size);
					if (av != &#x26;main_arena)
					victim->size |= NON_MAIN_ARENA;
				}
				/* Split */
				else
				{
					/* 获得剩余部分chunk指针 */
					remainder = chunk_at_offset (victim, nb);
					/* We cannot assume the unsorted list is empty and therefore
					   have to perform a complete insert here.  */
					/* 剩余部分作为新chunk加入到unsorted bins中 */
					bck = unsorted_chunks (av);
					fwd = bck->fd;
					if (__glibc_unlikely (fwd->bk != bck))
					{
						errstr = "malloc(): corrupted unsorted chunks";
						goto errout;
					}
					remainder->bk = bck;
					remainder->fd = fwd;
					bck->fd = remainder;
					fwd->bk = remainder;
					/* 若剩余部分大小属于large bin，则将fd_nextsize和bk_nextsize清零
					   因为这两个指针对于unsorted bin无用 */
					if (!in_smallbin_range (remainder_size))
					{
						remainder->fd_nextsize = NULL;
						remainder->bk_nextsize = NULL;
					}
					
					/* 设置各种标志位 */
					set_head (victim, nb | PREV_INUSE |
					(av != &#x26;main_arena ? NON_MAIN_ARENA : 0));
					set_head (remainder, remainder_size | PREV_INUSE);
					set_foot (remainder, remainder_size);
				}
				check_malloced_chunk (av, victim, nb);
				void *p = chunk2mem (victim);     //获取用户部分指针  
				alloc_perturb (p, bytes);
				return p;     //返回应用层
			}
		}


</code></pre>



## house of spirit

影响范围：2.23---至今

### 漏洞成因 <a href="#lou-dong-cheng-yin" id="lou-dong-cheng-yin"></a>

堆溢出写

### 漏洞利用

利用堆溢出，修改 `chunk size`，伪造出 `fake chunk`，然后通过堆的释放和排布（堆分水），控制 `fake chunk`。`house of spirit` 的操作思路有很多，比如可以按如下操作进行利用：

* 申请 `chunk A、chunk B、chunk C、chunk D`
* 对 `A` 写操作的时候溢出，修改 `B` 的 `size` 域，使其能包括 `chunk C`
* 释放 `B`，然后把 `B` 申请回来，再释放 `C`，则可以通过读写 `B` 来控制 `C` 的内容



## house of einherjar

#### 影响范围 <a href="#shi-yong-fan-wei-1" id="shi-yong-fan-wei-1"></a>

* `2.23`—— 至今
* 可分配大于处于 `unsortedbin` 的 `chunk`

### 漏洞成因 <a href="#lou-dong-cheng-yin-1" id="lou-dong-cheng-yin-1"></a>

溢出写、`off by one`、`off by null`

### 漏洞利用

利用 `off by null` 修改掉 `chunk` 的 `size` 域的 `P` 位，绕过 `unlink` 检查，在堆的后向合并过程中构造出 `chunk overlapping`。

* 申请 `chunk A、chunk B、chunk C、chunk D`，`chunk D` 用来做 `gap`，`chunk A、chunk C` 都要处于 `unsortedbin` 范围
* 释放 `A`，进入 `unsortedbin`
* 对 `B` 写操作的时候存在 `off by null`，修改了 `C` 的 `P` 位
* 释放 `C` 的时候，堆后向合并，直接把 `A、B、C` 三块内存合并为了一个 `chunk`，并放到了 `unsortedbin` 里面
* 读写合并后的大 `chunk` 可以操作 `chunk B` 的内容，`chunk B` 的头

可以理解为`unlink`攻击
