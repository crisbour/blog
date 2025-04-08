---
title: "Rust sys crate for .DLL"
author: "Cristian Bourceanu"
date: 2025-04-08T10:00:00Z
categories: [rust, windows, ffi]
---

# Making a *-sys crate for Windows library

Building a rust `*-sys` crate is already quite a standard process to reuse
stable library that expose a C-ABI and it's nicely explained in [kornel
rust-sys-crate blog](https://kornel.ski/rust-sys-crate). While this works great on Unix or with
statically linked library, if you had access to the source, a `.DLL` requires
[1] to load it at runtime.

[1] Or does it? I only managed to work with a `dll` using [libloading](https://docs.rs/libloading/latest/libloading/) crate,
while bindgen itself didn't manage to link it, no matter what flags I passed to
the rust compiler.

## Statically linked

Rust statically links the binaries compiled, however, not the dynamic library
which needs to be loaded at runtime. In the case of dynamic library which can be
linked at runtime because they are in the linker path, nothing else it's
required. However, if we want the executable to be self maintained, the dynamic
library path must be given.

My first attempt was to use the `OUT_DIR`, however this expands in an absolute
path which is written in the binary as a constant. Hence, changing the path of
the executable or transferring it on another machine, breaks it.


## Embed resource

The alternative I thought is to embed the `.dll` inside the executable using [embed-resource](https://docs.rs/embed-resource/latest/embed_resource/), extract
it at runtime and load it with `libloading`. This idea doesn't seem to have been
explored before, so I have no certainty yet that is works.

### Implemntation example

1. Add dependency to build step
```sh
cargo add --build embed-resource
```

2. Write the [windows resource file](https://learn.microsoft.com/en-us/windows/win32/menurc/about-resource-files) as follows in an `.rc` file.
```c
// Define the DLL as an RCDATA resource
IDR_MYDLL RCDATA "vendor_library.dll"
```

3. Embed the `dll` inside the executable:
```rust
fn main() {
    embed_resource::compile("dll_resource.rc", embed_resource::NONE)
        .manifest_optional()
        .unwrap();
}
```

4. Extract dll at runtime and load it with `libloading`:
```rust
use winapi::um::libloaderapi::{FindResourceW, LoadResource, SizeofResource, LockResource};
use winapi::um::winbase::FreeResource;
use winapi::um::winuser::RT_RCDATA;
use std::ptr;
use std::ffi::OsStr;
use std::os::windows::ffi::OsStrExt;
use libloading::{Library, Symbol};

unsafe fn extract_and_load_dll() -> Result<(), Box<dyn std::error::Error>> {
    // Convert resource name to wide string
    let resource_name = OsStr::new("IDR_MYDLL").encode_wide().chain(Some(0)).collect::<Vec<_>>();
    
    // Locate the resource
    let h_resource = FindResourceW(
        ptr::null_mut(),
        resource_name.as_ptr(),
        RT_RCDATA
    );
    
    // Load and lock the resource
    let h_global = LoadResource(ptr::null_mut(), h_resource);
    let dll_data = LockResource(h_global) as *const u8;
    let dll_size = SizeofResource(ptr::null_mut(), h_resource) as usize;
    
    // Write to temp file
    let temp_path = std::env::temp_dir().join("vendor_library.dll");
    std::fs::write(&temp_path, std::slice::from_raw_parts(dll_data, dll_size))?;
    
    // Load the DLL
    let lib = Library::new(&temp_path)?;
    let func: Symbol<unsafe extern "C" fn() -> i32> = lib.get(b"example_function")?;
    println!("Result: {}", func());
    
    // Cleanup (optional)
    FreeResource(h_global);
    std::fs::remove_file(temp_path)?;
    
    Ok(())
}
```

A more detailed example of how to work with embedded resources in Windows is
shown in [embed and extract binaries into/from exe as
resources](https://giodicanio.com/2024/03/25/embedding-and-extracting-binary-files-like-dlls-into-an-exe-as-resources/). In our case the resource type is `RCDATA` to describe a custom binary.
