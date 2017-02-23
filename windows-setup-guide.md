---
title: Windows Setup Guide
layout: page
---

MeTA can be built on Windows using the MinGW-w64 toolchain with gcc. We
currently only support using [MSYS2][msys2] as this makes fetching the
compiler and related libraries significantly easier than it would be
otherwise, and it tends to have very up-to-date packages relative to other
similar MinGW distributions.

This visual guide will help you get the MSYS2 environment installed and
updated, and then will show you how to compile and run the MeTA and its
unit tests. Do note that some of the steps *might* look slightly different
for you (different terminal output or prompts), but you should still be
able to follow this guide.

### Setting up MSYS2

To start, [download the installer][msys2] for MSYS2 from the linked
website. **Be sure to grab the x86_64 version**:

[![msys2 installer](images/windows/step01.png)](images/windows/step01.png)

Now, run the installer. Click **next** at the prompt:

[![msys2 installer 1](images/windows/step02.png)](images/windows/step02.png)

Use the default installation directory of `C:\msys64`. Click **next**:

[![msys2 installer 3](images/windows/step04.png)](images/windows/step04.png)

Set up shortcuts if you want. Click **next**:

[![msys2 installer 4](images/windows/step05.png)](images/windows/step05.png)

Leave the **Run MSYS2 64-bit now** option checked, and click **finish**:

[![msys2 installer 5](images/windows/step06.png)](images/windows/step06.png)

You should now see a MSYS2 terminal. You can tell that it is the MSYS2
terminal because it includes the purple text "MSYS" in the command prompt.

Run the following command:

{% highlight bash %}
pacman -Sy --needed bash pacman msys2-runtime
{% endhighlight %}

[![msys2 updating 1](images/windows/step07.png)](images/windows/step07.png)

Type **Y** at the yes/no prompt for installation, then hit enter:

[![msys2 updating 2](images/windows/step08.png)](images/windows/step08.png)

Once the installation finishes, close the terminal **by clicking the red X
in the corner**:

[![msys2 updating 3](images/windows/step09.png)](images/windows/step09.png)

Now, open a File Explorer window, and navigate to your `C:` drive:

[![msys2 updating 4](images/windows/step10.png)](images/windows/step10.png)

Find the `msys64` folder, and double click:

[![msys2 updating 5](images/windows/step11.png)](images/windows/step11.png)

Find the `msys2_shell` command and double click:

[![msys2 updating 6](images/windows/step12.png)](images/windows/step12.png)

You should be back at a MSYS terminal (notice the purple "MSYS" in the
prompt). Run the following command:

{% highlight bash %}
pacman -Syuu
{% endhighlight %}

[![msys2 updating 7](images/windows/step13.png)](images/windows/step13.png)

Type **Y** at the yes/no installation prompt and hit enter:

[![msys2 updating 8](images/windows/step14.png)](images/windows/step14.png)

When the install finishes, close the terminal **by clicking the red X in
the corner**:

[![msys2 updating 9](images/windows/step15.png)](images/windows/step15.png)

If you get a warning like this, just click **OK**:

[![msys2 updating 10](images/windows/step16.png)](images/windows/step16.png)

Launch the MSYS2 shell again by double clicking on `msys2_shell` in
`C:\msys64`. Run the following command:

{% highlight bash %}
pacman -Syu
{% endhighlight %}

[![msys2 updating 11](images/windows/step17.png)](images/windows/step17.png)

If you see any prompts about replacing certain packages with another
package (like pictured below), type **Y** and hit enter:

[![msys2 updating 12](images/windows/step18.png)](images/windows/step18.png)

Type **Y** at the yes/no installation prompt and hit enter:

[![msys2 updating 13](images/windows/step19.png)](images/windows/step19.png)

Once the install finishes, close the terminal by either clicking the red X
or by typing `exit`:

[![msys2 updating 14](images/windows/step20.png)](images/windows/step20.png)

Congratulations! MSYS2 should be set up and updated to the latest version
now.

### Getting MeTA's Build Dependencies

Now, let's get everything set up for MeTA. Navigate to `C:\msys64` and
double click on the `mingw64` application:

[![mingw updating 1](images/windows/step21.png)](images/windows/step21.png)

This will launch the "MINGW64" shell. You can tell that you are in the
MINGW64 shell by the purple "MINGW64" in the command prompt. **You will
want to use the MINGW64 shell for everything from now on.**

Just to be sure you've got everything up to date, run the following
command, typing **Y** and hitting enter at any prompts along the way:

{% highlight bash %}
pacman -Syu
{% endhighlight %}

[![mingw updating 2](images/windows/step22.png)](images/windows/step22.png)

Now, copy and paste the following command into your MINGW64 shell and then
hit enter:

{% highlight bash %}
pacman -Syu git make patch mingw-w64-x86_64-{gcc,cmake,icu,jemalloc,zlib} --force
{% endhighlight %}

[![mingw updating 3](images/windows/step23.png)](images/windows/step23.png)
[![mingw updating 4](images/windows/step24.png)](images/windows/step24.png)

At the yes/no installation prompt, type **Y** and hit enter:

[![mingw updating 5](images/windows/step25.png)](images/windows/step25.png)

Congratulations! The MinGW-w64 toolchain is now installed, along with all
the libraries and command-line tools that MeTA's build system uses.

[![mingw updating 6](images/windows/step26.png)](images/windows/step26.png)

### Getting and Compiling MeTA

Finally, we can download and compile MeTA and its unit tests. Copy and
paste the following command into a **MINGW64 shell**, and then press enter:

{% highlight bash %}
git clone https://github.com/meta-toolkit/meta.git
{% endhighlight %}

[![meta install 1](images/windows/step27.png)](images/windows/step27.png)

Move into the newly created `meta` directory by running the following
command:

{% highlight bash %}
cd meta
{% endhighlight %}

[![meta install 2](images/windows/step28.png)](images/windows/step28.png)

Now, fetch the submodules for MeTA by running the following command:

{% highlight bash %}
git submodule update --init --recursive
{% endhighlight %}

[![meta install 3](images/windows/step29.png)](images/windows/step29.png)

Now, make a build directory with the following command:

{% highlight bash %}
mkdir build
{% endhighlight %}

[![meta install 4](images/windows/step30.png)](images/windows/step30.png)

Move into this new directory with the following command:

{% highlight bash %}
cd build
{% endhighlight %}

[![meta install 5](images/windows/step31.png)](images/windows/step31.png)

Copy the sample configuration file into your build directory with the
following command:

{% highlight bash %}
cp ../config.toml .
{% endhighlight %}

[![meta install 6](images/windows/step32.png)](images/windows/step32.png)

Now, we will use `cmake` to generate our `Makefile`s by running the
following command **(every part of this is important, so please copy/paste
it)**:

{% highlight bash %}
cmake .. -G "MSYS Makefiles" -DCMAKE_BUILD_TYPE=Release
{% endhighlight %}

[![meta install 7](images/windows/step33.png)](images/windows/step33.png)

Once the `Makefile`s have been generated by `cmake`, we can run `make` to
do the actual building of MeTA:

[![meta install 8](images/windows/step34.png)](images/windows/step34.png)

Once the build process completes (with no errors!), you should be able to
run the unit tests:

{% highlight bash %}
./unit-test --reporter=spec
{% endhighlight %}

[![meta install 9](images/windows/step35.png)](images/windows/step35.png)

If everything passes, congratulations! MeTA seems to be working on your
system.

[![meta install 10](images/windows/step36.png)](images/windows/step36.png)

[msys2]: https://msys2.github.io/
