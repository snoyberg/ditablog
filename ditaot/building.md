Generating Output with the DITA-OT
==================================

The [DITA-OT](http://dita-ot.sourceforge.net/) is currently the defacto
standard for generating output from DITA. You can generate PDFs, HTML, and a
number of different outputs from it. However, it has a steep learning curve,
not just for customization, but also for normal usage.

In this post, we'll cover how to generate output and some of the general
terminology associated with it.

Transtypes
----------

The most important term for our purposes is __transtype__. In the DITA-OT, a
transtype is an output format. Some built-in options are xhtml and pdf2. It's
possible to provide additional transtypes via a __plugin__; for example, this
is how [SuiteHelp](http://www.suite-sol.com/pages/solutions/suitehelp.html)
works.

Under the surface, the entire DITA-OT build process is controlled by a tool
called Ant. Ant works by providing different __targets__. A transtype must
provide a target named `dita2`*`transtype`*. For example, if you search through
the DITA-OT, you'll find `dita2xhtml` and `dita2pdf2`. The SuiteHelp plugin
provides a `dita2suitehelp` target as well.

For basic output generation, we want to tell the DITA-OT two things: which
transtype we want to use, and the DITA map we're going to be generating.

Get the DITA-OT
---------------

Obviously you'll actually need to have the DITA-OT on your system to use it.
You can download from the Sourceforge download page. In general, it's
recommended to install the [full easy
install](http://sourceforge.net/projects/dita-ot/files/DITA-OT%20Stable%20Release/DITA%20Open%20Toolkit%201.5.4/DITA-OT1.5.4_full_easy_install_bin.zip/download)
version. 

Once you have the DITA-OT, you just need to unzip it on your system somewhere.
There is no requirement to install it in a traditional sense.

Get ready to run
----------------

The DITA-OT needs to be run from the command line. However, it requires a
number of special settings (known as __environment variables__) to be set.
There are two different choices for how to do this, depending on your operating
system.

*   On Windows, double-click on the `startcmd.bat` file. This will pop up a
    command window.

*   On Unix-like systems, including Linux and Mac OS X, open a terminal in the
    manner you normally would for your system. Then change directories to the
    DITA-OT folder, and run:

        DITA_HOME=`pwd` ./startcmd.sh

In either event, you will now have a command line ready that can run the
DITA-OT.

__Note__: You will also need to have Java installed on your system in order to
use the DITA-OT. Most operating systems today should already have it installed,
but if not, please do so now.

Run it
------

There are actually two options available at this point for generating output:
the command line invoker (CLI), and directly via Ant. The CLI is easier, but
does not offer as much flexibility as Ant. To generate some output, you would
run (on Windows):

    java -jar lib\dost.jar /transtype:xhtml /i:samples\taskbook.ditamap

Or on any other operating system:

    java -jar lib/dost.jar /transtype:xhtml /i:samples/taskbook.ditamap

Note the difference: on Windows, the __path separator__ is a backslash, whereas
on Unix-style systems, it's a forward slash.

What this line said is: I want you to run the Java program contained in the
dost.jar file, inside the lib folder. I want you to use the xhtml transtype,
and process the taskbook.ditamap input found in the samples subfolder. You can
replace xhtml with any other transtype (e.g., pdf2) and use your own content in
place of taskbook. There are also many other options available from the command
line, such as setting the output and temp folder. By default, the output will
be placed in the `out` folder.

If you want to use Ant directly, there are many different choices available to
you. I will describe the method I personally use, and what I recommend to our
clients. Create a file named `build.xml` in the folder *above* the DITA-OT. For
example, if you placed the DITA-OT in `c:\testing\DITA-OT1.5.4`, create a file
at `c:\testing\build.xml`. Inside this file, copy in the following content:

    <?xml version="1.0" encoding="UTF-8" ?>
    <project default="output" basedir=".">
        <import file="${basedir}/DITA-OT1.5.4/build.xml"/>
        <property name="ditamap" value="${basedir}/DITA-OT1.5.4/samples/taskbook.ditamap"/>

        <target name="output">
            <antcall target="dita2pdf2">
                <param name="args.input" value="${ditamap}"/>
                <param name="output.dir" value="${basedir}/output"/>
            </antcall>
        </target>
    </project>

A few explanations:

* I have used forward slashes for all the path names. This will work correctly
  on both Windows and Unix-like systems. Many documents recommend replacing
  this with `${file.separator}`, but for most operating systems today, the
  difference is irrelevant, and only causes the files to be much harder to read.

* As you may have noticed, an Ant build file is just a normal XML file. All
  normal XML rules apply.

* You need to change the `ditamap` property to refer to your actual map.

* Notice the `target` attribute on the `antcall`. In this case, I've used
  `dita2pdf2`. To change to a different transtype, simply replace `pdf2` with
  the transtype name (e.g., `suitehelp`).

* I've overridden the `output.dir` parameter. There are many other parameters
  available that can be overridden. Most of these are covered in the DITA-OT's
  docs.

In order to build, just type `ant`. Running `ant` without any parameters will
use the `build.xml` file in the current folder by default, and build the
default target. If you want, you can explicitly state both of those with:

    ant -f build.xml output
