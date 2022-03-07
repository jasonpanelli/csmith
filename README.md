# ![](./csmith.png)

## About

Csmith is a random generator of C programs. It's primary purpose is to find
compiler bugs with random programs, using differential testing as the
test oracle.

Csmith can be used outside of the field of compiler testing.
If your application needs a test suite of C programs and you don't bother to
write them, feel free to give Csmith a try.

Csmith outputs C programs free of undefined behaviors (believe us, that's
not trivial), and the statistics of each generated program.

## About HLSCsmith

This version of Csmith has been extended to produce random programs that are
compatible with the Intel HLS Compiler and optionally contain HLS-specific
compiler directives.

## Install Csmith

You can install Csmith from tarballs downloaded from [here (coming soon)](doc/releases.md),
or you can build it from the source. The following commands
apply to Ubuntu.

```
git clone https://github.com/csmith-project/csmith.git
cd csmith
sudo apt install g++ cmake m4
cmake -DCMAKE_INSTALL_PREFIX=<INSTALL-PREFIX> .
make && make install
```

Please see specific instructions for [building on
Windows](doc/build-csmith-on-windows.md).

## Use Csmith

Suppose Csmith is installed to `$HOME/csmith` locally. You can simply
generate, compile, and execute a test case by:

```bash
export PATH=$PATH:$HOME/csmith/bin
csmith > random1.c
gcc random1.c -I$HOME/csmith/include -o random1
./random1
```

To add differential testing into the picture, we need to install another
compiler, e.g., another version of **gcc** or **clang**. And repeat the process of:

```
csmith > random2.c
gcc random2.c -I$HOME/csmith/include -o random2_gcc
clang random2.c -I$HOME/csmith/include -o random2_clang
./random2_gcc > gcc_output.txt
./random2_clang > clang_output.txt
```

If there is any difference in `gcc_output.txt` and `clang_output.txt`,
aha, you have found a bug in either **gcc** or **clang**, or, in the
unlikely case, a bug in Csmith itself.

You could write scripts in your favorite language to repeat
the above process to amplify the power of random differential testing.

The generate programs might contain infinite loops. The best practice is
to apply timeout to their executions.

Use `csmith -h` or `csmith -hh` to see lists of command line options that you
can pass to Csmith and customize the random generation.

Here is a slightly outdated but still relevant document about
[using Csmith for compiler testing](http://embed.cs.utah.edu/csmith/using.html).

## Run Csmith with HLS

### Run Csmith with probabilities file 
You must specify the probability file we provide and use all flags shown here when generating HLS-compatible programs. These flags are required in all the following examples as well.
```
csmith --probability-configuration prob.txt --no-argc --max-array-dim 3 --max-funcs 5 --max-expr-complexity 2 --no-float --no-embedded-assigns --max-block-depth 2 --no-unions --no-packed-struct --no-const-pointers --no-pointers --strict-const-arrays > random.c
```

### Run Csmith with HLS mode
We have added an HLS mode that creates a component function to synthesize. It is required when generating programs that will be compiled to target an FPGA.
```
csmith --hls-mode > random.c
```

### Run Csmith with component function 
To randomly label functions as components, use `--component-function`. You can modify the value of `component_function_prob=<number between 0-100>` in prob.txt or specify it on the command line.
```
csmith --component-function > random.c
csmith --component-function --component-function-prob 50 > random.c
```

### Run Csmith with loop pragmas 
To add loop pragmas, use `--loop-pragmas`. You can modify the value of `loop_pragma_prob=<number between 0-100>` in prob.txt or specify it on the command line.
```
csmith --loop-pragmas > random.c
csmith --loop-pragmas --loop-pragma-prob 50 > random.c
```

### Run Csmith with static variables
To add static variables, use `--static-vars`.  You can modify the value of `static_var_prob=<number between 0-100>` in prob.txt or specify it on the command line.
```
csmith --static-vars > random.c
csmith --static-vars --static-var-prob 50 > random.c
```

### Compile Csmith-generated programs with HLS Compiler 
Command has the following structure. 
```
dpcpp -Wno-narrowing -fintelfpga -march=x86-64 [filename] -o [output executable name].exe -I[path to hls-include]
```

An example command. 
```
dpcpp -Wno-narrowing -fintelfpga -march=x86-64 random.c -o random.exe -Ihls-include
```

Using the Intel HLS Compiler (i++) directly. This is how we test our programs locally.
```
i++ -Wno-narrowing -march=x86-64 -IC:\csmith\runtime -IC:\intelFPGA_pro\21.4\hls\include random.c  -o random.exe
```

Compiling to target an FPGA. This has only been tested locally with i++.
```
i++ -Wno-narrowing -march=CycloneV -IC:\csmith\runtime -IC:\intelFPGA_pro\21.4\hls\include random.c  -o random.exe
```

### Run Compiled Program 
Run the generated executable. 
```
./random.exe
```

## History

Csmith was originally developed at the University of Utah by:

* [Xuejun Yang](https://github.com/jxyang)
* [Yang Chen](https://github.com/chenyang78)
* [Eric Eide](https://github.com/eeide)
* [John Regehr](https://github.com/regehr)

as part of a research project on compiler testing. The research is best
summarized by our paper
[Finding and Understanding Bugs in C Compilers](https://www.cs.utah.edu/~regehr/papers/pldi11-preprint.pdf).
More research info can be found
[here](http://embed.cs.utah.edu/csmith/).

Csmith was open sourced in 2009. We try to keep maintaining it as an open source
project using discretionary times. As much, the response to bug reports or
feature requests might be delayed.

## Community

Please use github [issues](https://github.com/csmith-project/csmith/issues/new)
to report bugs or suggestions.

We have a mailing list for discussing Csmith.
Please visit [here](http://www.flux.utah.edu/mailman/listinfo/csmith-dev) to subscribe.


