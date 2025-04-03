---
description: 特别鸣谢：chatgpt
---

# WindowsAPI

### 1. GetDlgItemTextA

**函数原型：**

```c
UINT GetDlgItemTextA(
  HWND   hDlg,        // 对话框句柄
  int    nIDDlgItem,  // 控件ID
  LPSTR  lpString,    // 存储文本的缓冲区指针
  int    nMaxCount    // 缓冲区最大字符数
);
```

**参数说明：**

* **hDlg**：对话框窗口的句柄，标识当前对话框。
* **nIDDlgItem**：对话框内控件的 ID。
* **lpString**：指向缓冲区的指针，用于接收控件内的文本。
* **nMaxCount**：缓冲区允许接收的最大字符数。

**返回值：**\
返回复制到缓冲区中的字符数（不包括结尾的 NULL 字符），如果出错返回 0。

**汇编中的表现（以 x86 为例）：**

调用时通常类似如下过程（参数从右到左压栈）：

```asm
push    nMaxCount        ; 最大字符数
push    lpString         ; 缓冲区地址
push    nIDDlgItem       ; 控件ID
push    hDlg             ; 对话框句柄
call    GetDlgItemTextA
; 返回值在 EAX 中，表示实际复制的字符数
```

***

### 2. SetDlgItemTextA

**函数原型：**

```c
BOOL SetDlgItemTextA(
  HWND   hDlg,       // 对话框句柄
  int    nIDDlgItem, // 控件ID
  LPCSTR lpString    // 要设置的文本字符串
);
```

**参数说明：**

* **hDlg**：对话框句柄。
* **nIDDlgItem**：对话框内控件的 ID。
* **lpString**：指向要设置的字符串的指针。

**返回值：**\
非零表示成功，零表示失败。

**汇编表示：**

```asm
push    lpString
push    nIDDlgItem
push    hDlg
call    SetDlgItemTextA
; 返回值在 EAX 中
```

***

### 3. MessageBoxA

**函数原型：**

```c
int MessageBoxA(
  HWND    hWnd,        // 父窗口句柄
  LPCSTR  lpText,      // 消息文本
  LPCSTR  lpCaption,   // 消息框标题
  UINT    uType        // 消息框样式
);
```

**参数说明：**

* **hWnd**：父窗口句柄，可以为 NULL。
* **lpText**：消息文本字符串。
* **lpCaption**：消息框标题字符串。
* **uType**：消息框的样式标志（例如按钮组合、图标等）。

**返回值：**\
返回用户点击按钮的标识符，如 IDOK、IDCANCEL 等。

**汇编表示：**

```asm
push    uType
push    lpCaption
push    lpText
push    hWnd
call    MessageBoxA
; 返回值在 EAX 中
```

***

### 4. GetWindowTextA

**函数原型：**

```c
int GetWindowTextA(
  HWND   hWnd,         // 窗口句柄
  LPSTR  lpString,     // 缓冲区地址
  int    nMaxCount     // 缓冲区大小
);
```

**参数说明：**

* **hWnd**：目标窗口句柄。
* **lpString**：用于接收窗口标题文本的缓冲区指针。
* **nMaxCount**：缓冲区的大小（字符数）。

**返回值：**\
返回复制到缓冲区的字符数，不包含终止符；如果窗口没有文本或出错返回 0。

**汇编表示：**

```asm
push    nMaxCount
push    lpString
push    hWnd
call    GetWindowTextA
; 返回值在 EAX 中
```

***

### 5. SendMessageA

**函数原型：**

```c
LRESULT SendMessageA(
  HWND   hWnd,      // 目标窗口句柄
  UINT   Msg,       // 消息编号
  WPARAM wParam,    // 消息附带的参数
  LPARAM lParam     // 消息附带的参数
);
```

**参数说明：**

* **hWnd**：接收消息的窗口句柄。
* **Msg**：消息标识，如 WM\_COMMAND、WM\_SETTEXT 等。
* **wParam** 与 **lParam**：根据具体消息不同携带不同的信息。

**返回值：**\
返回值根据具体消息定义而异。

**汇编表示：**

```asm
push    lParam
push    wParam
push    Msg
push    hWnd
call    SendMessageA
; 返回值在 EAX 中
```

***

### 6. VirtualAlloc

* **功能描述**：在进程的虚拟地址空间中分配一块内存区域，可用于动态内存申请。
*   **汇编调用示例**：

    ```asm
    push    flAllocationType   ; 内存分配类型（如 MEM_COMMIT | MEM_RESERVE）
    push    flProtect          ; 内存保护属性（如 PAGE_READWRITE）
    push    dwSize             ; 分配内存大小
    push    lpAddress          ; 指定起始地址或 NULL
    call    VirtualAlloc
    ; 返回值在 EAX 中，为分配内存的起始地址
    ```

***

### 7. VirtualFree

* **功能描述**：释放之前通过 VirtualAlloc 分配的虚拟内存区域。
*   **汇编调用示例**：

    ```asm
    push    dwFreeType         ; 释放类型（如 MEM_RELEASE）
    push    dwSize             ; 释放内存的大小（对于 MEM_RELEASE 必须为 0）
    push    lpAddress          ; 要释放内存区域的起始地址
    call    VirtualFree
    ; 返回值在 EAX 中，非零表示成功
    ```

***

### 8. CreateFileA

* **功能描述**：打开或者创建一个文件或设备，返回文件句柄，用于后续的文件读写操作。
*   **汇编调用示例**：

    ```asm
    push    dwFlagsAndAttributes ; 文件属性和标志
    push    dwCreationDisposition; 文件创建行为（如 OPEN_EXISTING）
    push    dwShareMode          ; 文件共享模式
    push    lpSecurityAttributes ; 安全属性指针或 NULL
    push    dwDesiredAccess      ; 文件访问权限（如 GENERIC_READ）
    push    lpFileName           ; 文件名字符串指针
    call    CreateFileA
    ; 返回值在 EAX 中，为文件句柄（INVALID_HANDLE_VALUE 表示失败）
    ```

***

### 9. CreateFileW

* **功能描述**：与 CreateFileA 类似，但使用 Unicode 字符串（宽字符）表示文件名。
*   **汇编调用示例**：

    ```asm
    push    dwFlagsAndAttributes ; 文件属性和标志
    push    dwCreationDisposition; 文件创建行为
    push    dwShareMode          ; 文件共享模式
    push    lpSecurityAttributes ; 安全属性指针或 NULL
    push    dwDesiredAccess      ; 文件访问权限
    push    lpFileNameW          ; Unicode 文件名字符串指针
    call    CreateFileW
    ; 返回值在 EAX 中，为文件句柄
    ```

***

### 10. ReadFile

* **功能描述**：从指定的文件句柄中读取数据到缓冲区。
*   **汇编调用示例**：

    ```asm
    push    lpOverlapped       ; 重叠结构指针或 NULL
    push    lpNumberOfBytesRead; 实际读取字节数存储地址
    push    nNumberOfBytesToRead; 期望读取的字节数
    push    lpBuffer           ; 数据接收缓冲区地址
    push    hFile              ; 文件句柄
    call    ReadFile
    ; 返回值在 EAX 中，非零表示成功
    ```

***

### 11. WriteFile

* **功能描述**：向指定的文件句柄写入数据。
*   **汇编调用示例**：

    ```asm
    push    lpOverlapped       ; 重叠结构指针或 NULL
    push    lpNumberOfBytesWritten; 实际写入字节数存储地址
    push    nNumberOfBytesToWrite; 要写入的字节数
    push    lpBuffer           ; 数据缓冲区地址
    push    hFile              ; 文件句柄
    call    WriteFile
    ; 返回值在 EAX 中，非零表示成功
    ```

***

### 12. CloseHandle

* **功能描述**：关闭内核对象句柄，如文件、线程、进程等。
*   **汇编调用示例**：

    ```asm
    push    hObject           ; 要关闭的句柄
    call    CloseHandle
    ; 返回值在 EAX 中，非零表示成功
    ```

***

### 13. LoadLibraryA

* **功能描述**：加载指定的动态链接库（DLL），返回模块句柄，用于后续的函数地址获取等操作。
*   **汇编调用示例**：

    ```asm
    push    lpLibFileName     ; DLL 文件名的 ASCII 字符串指针
    call    LoadLibraryA
    ; 返回值在 EAX 中，为模块句柄
    ```

***

### 14. GetProcAddress

* **功能描述**：获取指定模块中某个导出函数的地址。
*   **汇编调用示例**：

    ```asm
    push    lpProcName        ; 函数名字符串指针或序号（低16位有效）
    push    hModule           ; 模块句柄
    call    GetProcAddress
    ; 返回值在 EAX 中，为函数地址
    ```

***

### 15. RegOpenKeyExA

* **功能描述**：打开注册表项，获取一个句柄用于后续的注册表操作。
*   **汇编调用示例**：

    ```asm
    push    ulOptions         ; 保留参数，一般为 0
    push    samDesired        ; 打开项的访问权限标志
    push    lpSubKey          ; 子项名称字符串指针
    push    hKey              ; 打开的根键
    call    RegOpenKeyExA
    ; 返回值在 EAX 中，ERROR_SUCCESS 表示成功
    ```

***

### 16. RegQueryValueExA

* **功能描述**：查询指定注册表项中某个值的信息（如数据、类型等）。
*   **汇编调用示例**：

    ```asm
    push    lpcbData          ; 数据缓冲区大小指针
    push    lpData            ; 数据缓冲区指针
    push    lpReserved        ; 保留参数，一般为 NULL
    push    lpType            ; 数据类型存储地址
    push    lpValueName       ; 值名称字符串指针
    push    hKey              ; 注册表项句柄
    call    RegQueryValueExA
    ; 返回值在 EAX 中，ERROR_SUCCESS 表示成功
    ```

***

### 17. RegSetValueExA

* **功能描述**：在指定的注册表项中设置一个值，包括数据和数据类型。
*   **汇编调用示例**：

    ```asm
    push    cbData            ; 数据大小（以字节为单位）
    push    lpData            ; 指向数据的指针
    push    dwType            ; 数据类型（如 REG_SZ）
    push    lpValueName       ; 值名称字符串指针
    push    hKey              ; 注册表项句柄
    call    RegSetValueExA
    ; 返回值在 EAX 中，ERROR_SUCCESS 表示成功
    ```



#### 汇编调用约定及注意事项

1. **调用约定**：\
   以上大部分 API 使用 stdcall 调用约定，所以函数调用后内部会负责清理栈。如果你看到函数调用后没有额外的栈指针调整操作（例如 `add esp, X`），这通常是因为调用的函数自身完成了栈清理。
2. **参数传递**：\
   参数按照从右向左的顺序压栈。例如在调用 `GetDlgItemTextA` 时，先将 `nMaxCount` 压栈，最后压入 `hDlg`。
3. **返回值处理**：\
   函数的返回值通常存放在 EAX 寄存器中，在逆向时你可以观察 EAX 的值来了解函数执行结果。
4. **字符串处理**：\
   很多函数涉及字符串操作（例如获取或设置控件文本），需要特别注意内存地址、缓冲区大小以及字符串结束符（NULL）的处理。在汇编中这部分通常由调用者准备好内存地址，然后由 API 填充或读取内容。
