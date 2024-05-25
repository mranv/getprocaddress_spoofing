# SwapExportOrdinals

## Overview

The `swapexportordinals` function swaps the ordinals of two specified exported functions within a loaded DLL. This is useful in various scenarios, such as reverse engineering or modifying the behavior of certain functions within a DLL without altering the actual DLL file.

## Requirements

- Rust programming language
- `winapi` crate for Windows API interactions

## Dependencies

Ensure the following dependencies are added to your `Cargo.toml`:

```toml
[dependencies]
winapi = { version = "0.3", features = ["errhandlingapi", "winnt", "ctypes", "libloaderapi", "synchapi", "processthreadsapi", "memoryapi", "winuser"] }
md5 = "0.7"
```

## Functionality

### `swapexportordinals`

The `swapexportordinals` function takes the following parameters:
- `dllbase`: A pointer to the base address of the loaded DLL.
- `prochandle`: A handle to the process where the DLL is loaded.
- `swap1`: The name of the first function to swap.
- `swap2`: The name of the second function to swap.

### How it Works

1. **Read DOS Header**: The function reads the DOS header of the DLL to verify it's a valid PE file.
2. **Read NT Headers**: It then reads the NT headers to get the details about the sections and the export table.
3. **Read Sections**: It reads the section headers to locate the export table.
4. **Read Export Table**: The export table is read to find the addresses and ordinals of the exported functions.
5. **Find Functions**: The function iterates over the export table to find the specified functions (`swap1` and `swap2`).
6. **Swap Ordinals**: The ordinals of the specified functions are swapped using `ReadProcessMemory` and `WriteProcessMemory`.

### Example Usage

The example provided loads the `User32.dll` library and swaps the ordinals of `MessageBoxA` and `ChangeMenuW`. After the swap, it attempts to call `ChangeMenuW`, which should now execute the code for `MessageBoxA`.

```rust
fn main() {
    unsafe {
        let modulehandle = LoadLibraryA("User32\0".as_bytes().as_ptr() as *const i8);
        swapexportordinals(modulehandle as *mut c_void, GetCurrentProcess(), "MessageBoxA".to_string(), "ChangeMenuW".to_string());

        let mbox = GetProcAddress(modulehandle, "ChangeMenuW\0".as_bytes().as_ptr() as *const i8);

        let mbox2 = std::mem::transmute::<
        *mut c_void,fn(*mut c_void, *const i8, *const i8, u32) -> i32>(mbox as *mut c_void);
   
        mbox2(std::ptr::null_mut(),
        "Hi\0".as_bytes().as_ptr() as *const i8,
        "Hi\0".as_bytes().as_ptr() as *const i8,0);
    }
}
```

## Important Notes

- This code manipulates the memory of a running process and uses unsafe operations. Ensure you understand the implications and potential risks of running such code.
- The provided example works on the assumption that the functions `FillStructureFromMemory` and `ReadStringFromMemory` are implemented correctly elsewhere in your code.

## Disclaimer

This code is intended for educational purposes and should be used responsibly. Manipulating process memory can cause instability or crashes and should be done with caution. Always ensure you have permission to modify the target processes and understand the legal implications of doing so.
