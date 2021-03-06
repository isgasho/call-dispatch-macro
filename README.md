# call-dispatch-macro

[![Actions Status](https://github.com/wangrunji0408/call-dispatch-macro/workflows/CI/badge.svg)](https://github.com/wangrunji0408/call-dispatch-macro/actions)

Generate function call dispatcher in Rust procedural macro.

🚧 Working In Progress 🚧

## Usage

```rust
use call_dispatch_macro::call_dispatch;

// Define a struct.
struct Syscall;

// Put `call_dispatch` attribute on the impl block.
#[call_dispatch]
impl Syscall {
    // Mark the dispatcher function.
    #[dispatcher(match_arm_prefix = "sys")]
    fn dispatch(&mut self, num: u32, args: [usize; 6]) -> Option<i32> {
        // The body will be rewritten by the macro.
        panic!("code generated by macro")
        // ======== Generated Code Start ========
        match num {
            sys::SYS_READ => Some(self.sys_read(args[0] as _, args[1] as _, args[2] as _)),
            sys::SYS_WRITE => Some(self.sys_write(args[0] as _, args[1] as _, args[2] as _)),
            _ => None,
        }
        // ========= Generated Code End =========
    }

    // Mark the function to be dispatched.
    #[call]
    fn sys_read(&mut self, fd: i32, buf: *mut u8, len: usize) -> i32 {
        println!("sys_read: fd={:?}, buf=({:?}; {:?})", fd, buf, len);
        1
    }
    #[call]
    fn sys_write(&mut self, fd: i32, buf: *const u8, len: usize) -> i32 {
        println!("sys_write: fd={:?}, buf=({:?}; {:?})", fd, buf, len);
        2
    }
}

// Define constants for match number.
mod sys {
    pub const SYS_READ: u32 = 1;
    pub const SYS_WRITE: u32 = 2;
}

fn main() {
    let mut syscall = Syscall;
    assert_eq!(syscall.dispatch(0, [1, 2, 3, 4, 5, 6]), None);
    assert_eq!(syscall.dispatch(1, [1, 2, 3, 4, 5, 6]), Some(1));
    assert_eq!(syscall.dispatch(2, [1, 2, 3, 4, 5, 6]), Some(2));
}
```
