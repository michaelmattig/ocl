# ocl

#### [Documentation](http://doc.cogciprocate.com/ocl/) | [Change Log](https://github.com/cogciprocate/ocl/blob/master/RELEASES.md)

[![](http://meritbadge.herokuapp.com/ocl)](https://crates.io/crates/ocl)


OpenCL&trade; interfaces for Rust. Makes easy to use the most common features
of OpenCL. All interfaces are virtually zero-cost and perform on a par with
any C++ libraries.

Interfaces are still mildly unstable. Changes are now being documented in
[RELEASES.md](https://github.com/cogciprocate/ocl/blob/master/RELEASES.md).


## Goals

To provide:
- A simple and intuitive interface with OpenCL devices
- The full functionality of the OpenCL C ABI
- An absolute minimum of boilerplate
- As close as possible to zero performance overhead


## Platforms

Tested so far only on Linux (and probably OS X - need confirmation). Windows
support looks imminent. Please [provide
feedback](https://github.com/cogciprocate/ocl_rust/issues) about failures and
successes on your platform.


## Installation

Ensure that an OpenCL library is installed for your platform and that `clinfo`
or some other diagnostic command will run.

Add:

```
[dependencies] ocl = "0.5"
```

to your project's `Cargo.toml`.


## Example 

From 'examples/trivial.rs':
```
extern crate ocl;
use ocl::{ProQue, SimpleDims, Buffer};

fn main() {
    // Define a kernel:
    let src = r#"
        __kernel void multiply(__global float* buffer, float coeff) {
            buffer[get_global_id(0)] *= coeff;
        }
    "#;

    // Create an all-in-one context, program, and command queue:
    let ocl_pq = ProQue::builder().src(src).build().unwrap();

    // Set our work dimensions / data set size to something arbitrary:
    let dims = SimpleDims::One(500000);

    // Create a 'Buffer' with a built-in vector and initialize it with random 
    // floats between 0.0 and 20.0:
    let mut buffer: Buffer<f32> = Buffer::with_vec_scrambled(
         (0.0, 20.0), &dims, &ocl_pq.queue());

    // Create a kernel with arguments matching those in the kernel:
    let kern = ocl_pq.create_kernel("multiply", dims.work_dims()).unwrap()
        .arg_buf(&buffer)
        .arg_scl(10.0f32);

    // // Enqueue kernel:
    kern.enqueue(None, None);

    // // Read results from the device into buffer's local vector:
    buffer.fill_vec().unwrap();

    // // Print a result:
    println!("The value at index [{}] is '{}'!", 200007, buffer[200007]);
}

```


### Taking Requests

Want to bring your OpenCL-ness to Rust but can't find the functionality you
need? File an [issue](https://github.com/cogciprocate/ocl_rust/issues) and
let us know what should come next.


### Help

*If troubleshooting your OpenCL drivers:* check that `/usr/lib/libOpenCL.so.1`
exists. Go ahead and link `/usr/lib/libOpenCL.so -> libOpenCL.so.1` just in
case it's not already (AMD drivers sometimes don't create this link).  Intel
and AMD also have OpenCL libraries for your CPU if you're having trouble
getting your GPU to work (intel: [windows](http://registrationcenter.intel.com
/irc_nas/5198/opencl_runtime_15.1_x64_setup.msi), [linux](http://registrationc
enter.intel.com/irc_nas/5193/opencl_runtime_15.1_x64_5.0.0.57.tgz)).

Please ask questions and provide feedback by opening an
[issue](https://github.com/cogciprocate/ocl_rust/issues).

<br/><br/>

*“OpenCL and the OpenCL logo are trademarks of Apple Inc. used by permission
by Khronos.”*
