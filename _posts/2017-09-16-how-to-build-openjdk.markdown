---
layout:     post
title:      "[EN] How to build OpenJDK"
subtitle:   "OpenJDK"
date:       2017-09-16 02:59:59
author:     "Elve Xu"
header-img: "img/in-post/code-bg.jpg"
catalog: true
tags:
    - Java
    - openjdk
    - Build Document
---

> Code is Not Only Code

<p><img src="https://github.com/misselvexu/misselvexu.github.io/blob/master/img/in-post/openjdk.png?raw=true" alt="OpenJDK" title="" /></p>

<h1>OpenJDK Build README</h1>

<hr />

<p><a name="introduction"></a></p>

<h2>Introduction</h2>

<p>This README file contains build instructions for the
<a href="http://openjdk.java.net">OpenJDK</a>. Building the source code for the OpenJDK
requires a certain degree of technical expertise.</p>

<h3>!!!!!!!!!!!!!!! THIS IS A MAJOR RE-WRITE of this document. !!!!!!!!!!!!!</h3>

<p>Some Headlines:</p>

<ul>
<li>The build is now a "<code>configure &amp;&amp; make</code>" style build</li>
<li>Any GNU make 3.81 or newer should work, except on Windows where 4.0 or newer
is recommended.</li>
<li>The build should scale, i.e. more processors should cause the build to be
done in less wall-clock time</li>
<li>Nested or recursive make invocations have been significantly reduced,
as has the total fork/exec or spawning of sub processes during the build</li>
<li>Windows MKS usage is no longer supported</li>
<li>Windows Visual Studio <code>vsvars*.bat</code> and <code>vcvars*.bat</code> files are run
automatically</li>
<li>Ant is no longer used when building the OpenJDK</li>
<li>Use of ALT_* environment variables for configuring the build is no longer
supported</li>
</ul>

<hr />

<h2>Contents</h2>

<ul>
<li><a href="#introduction">Introduction</a></li>
<li><a href="#hg">Use of Mercurial</a>
<ul>
<li><a href="#get_source">Getting the Source</a></li>
<li><a href="#repositories">Repositories</a></li>
</ul></li>
<li><a href="#building">Building</a>
<ul>
<li><a href="#setup">System Setup</a>
<ul>
<li><a href="#linux">Linux</a></li>
<li><a href="#solaris">Solaris</a></li>
<li><a href="#macosx">Mac OS X</a></li>
<li><a href="#windows">Windows</a></li>
</ul></li>
<li><a href="#configure">Configure</a></li>
<li><a href="#make">Make</a></li>
</ul></li>
<li><a href="#testing">Testing</a></li>
</ul>

<hr />

<ul>
<li><a href="#hints">Appendix A: Hints and Tips</a>
<ul>
<li><a href="#faq">FAQ</a></li>
<li><a href="#performance">Build Performance Tips</a></li>
<li><a href="#troubleshooting">Troubleshooting</a></li>
</ul></li>
<li><a href="#gmake">Appendix B: GNU Make Information</a></li>
<li><a href="#buildenvironments">Appendix C: Build Environments</a></li>
</ul>

<hr />

<p><a name="hg"></a></p>

<h2>Use of Mercurial</h2>

<p>The OpenJDK sources are maintained with the revision control system
<a href="http://mercurial.selenic.com/wiki/Mercurial">Mercurial</a>. If you are new to
Mercurial, please see the <a href="http://mercurial.selenic.com/wiki/
BeginnersGuides">Beginner Guides</a> or refer to the <a href="http://hgbook.red-bean.com/">Mercurial Book</a>.
The first few chapters of the book provide an excellent overview of Mercurial,
what it is and how it works.</p>

<p>For using Mercurial with the OpenJDK refer to the <a href="http://openjdk.java.net/guide/
repositories.html#installConfig">Developer Guide: Installing
and Configuring Mercurial</a> section for more information.</p>

<p><a name="get_source"></a></p>

<h3>Getting the Source</h3>

<p>To get the entire set of OpenJDK Mercurial repositories use the script
<code>get_source.sh</code> located in the root repository:</p>

<pre><code>  hg clone http://hg.openjdk.java.net/jdk9/jdk9 YourOpenJDK
  cd YourOpenJDK
  bash ./get_source.sh
</code></pre>

<p>Once you have all the repositories, keep in mind that each repository is its
own independent repository. You can also re-run <code>./get_source.sh</code> anytime to
pull over all the latest changesets in all the repositories. This set of
nested repositories has been given the term "forest" and there are various
ways to apply the same <code>hg</code> command to each of the repositories. For
example, the script <code>make/scripts/hgforest.sh</code> can be used to repeat the
same <code>hg</code> command on every repository, e.g.</p>

<pre><code>  cd YourOpenJDK
  bash ./make/scripts/hgforest.sh status
</code></pre>

<p><a name="repositories"></a></p>

<h3>Repositories</h3>

<p>The set of repositories and what they contain:</p>

<ul>
<li><strong>. (root)</strong> contains common configure and makefile logic</li>
<li><strong>hotspot</strong> contains source code and make files for building the OpenJDK
Hotspot Virtual Machine</li>
<li><strong>langtools</strong> contains source code for the OpenJDK javac and language tools</li>
<li><strong>jdk</strong> contains source code and make files for building the OpenJDK runtime
libraries and misc files</li>
<li><strong>jaxp</strong> contains source code for the OpenJDK JAXP functionality</li>
<li><strong>jaxws</strong> contains source code for the OpenJDK JAX-WS functionality</li>
<li><strong>corba</strong> contains source code for the OpenJDK Corba functionality</li>
<li><strong>nashorn</strong> contains source code for the OpenJDK JavaScript implementation</li>
</ul>

<h3>Repository Source Guidelines</h3>

<p>There are some very basic guidelines:</p>

<ul>
<li>Use of whitespace in source files (.java, .c, .h, .cpp, and .hpp files) is
restricted. No TABs, no trailing whitespace on lines, and files should not
terminate in more than one blank line.</li>
<li>Files with execute permissions should not be added to the source
repositories.</li>
<li>All generated files need to be kept isolated from the files maintained or
managed by the source control system. The standard area for generated files
is the top level <code>build/</code> directory.</li>
<li>The default build process should be to build the product and nothing else,
in one form, e.g. a product (optimized), debug (non-optimized, -g plus
assert logic), or fastdebug (optimized, -g plus assert logic).</li>
<li>The <code>.hgignore</code> file in each repository must exist and should include
<code>^build/</code>, <code>^dist/</code> and optionally any <code>nbproject/private</code> directories. <strong>It
should NEVER</strong> include anything in the <code>src/</code> or <code>test/</code> or any managed
directory area of a repository.</li>
<li>Directory names and file names should never contain blanks or non-printing
characters.</li>
<li>Generated source or binary files should NEVER be added to the repository
(that includes <code>javah</code> output). There are some exceptions to this rule, in
particular with some of the generated configure scripts.</li>
<li>Files not needed for typical building or testing of the repository should
not be added to the repository.</li>
</ul>

<hr />

<p><a name="building"></a></p>

<h2>Building</h2>

<p>The very first step in building the OpenJDK is making sure the system itself
has everything it needs to do OpenJDK builds. Once a system is setup, it
generally doesn't need to be done again.</p>

<p>Building the OpenJDK is now done with running a <code>configure</code> script which will
try and find and verify you have everything you need, followed by running
<code>make</code>, e.g.</p>

<blockquote>
  <p><strong><code>bash ./configure</code></strong> <br />
 <strong><code>make all</code></strong></p>
</blockquote>

<p>Where possible the <code>configure</code> script will attempt to located the various
components in the default locations or via component specific variable
settings. When the normal defaults fail or components cannot be found,
additional <code>configure</code> options may be necessary to help <code>configure</code> find the
necessary tools for the build, or you may need to re-visit the setup of your
system due to missing software packages.</p>

<p><strong>NOTE:</strong> The <code>configure</code> script file does not have execute permissions and
will need to be explicitly run with <code>bash</code>, see the source guidelines.</p>

<hr />

<p><a name="setup"></a></p>

<h3>System Setup</h3>

<p>Before even attempting to use a system to build the OpenJDK there are some very
basic system setups needed. For all systems:</p>

<ul>
<li><p>Be sure the GNU make utility is version 3.81 (4.0 on windows) or newer, e.g.
run "<code>make -version</code>"</p>

<p><a name="bootjdk"></a></p></li>
<li><p>Install a Bootstrap JDK. All OpenJDK builds require access to a previously
released JDK called the <em>bootstrap JDK</em> or <em>boot JDK.</em> The general rule is
that the bootstrap JDK must be an instance of the previous major release of
the JDK. In addition, there may be a requirement to use a release at or
beyond a particular update level.</p>

<p><strong><em>Building JDK 9 requires JDK 8. JDK 9 developers should not use JDK 9 as
the boot JDK, to ensure that JDK 9 dependencies are not introduced into the
parts of the system that are built with JDK 8.</em></strong></p>

<p>The JDK 8 binaries can be downloaded from Oracle's <a href="http://www.oracle.com/technetwork/java/javase/downloads/index.html">JDK 8 download
site</a>.
For build performance reasons it is very important that this bootstrap JDK
be made available on the local disk of the machine doing the build. You
should add its <code>bin</code> directory to the <code>PATH</code> environment variable. If
<code>configure</code> has any issues finding this JDK, you may need to use the
<code>configure</code> option <code>--with-boot-jdk</code>.</p></li>
<li><p>Ensure that GNU make, the Bootstrap JDK, and the compilers are all in your
PATH environment variable.</p></li>
</ul>

<p>And for specific systems:</p>

<ul>
<li><p><strong>Linux</strong></p>

<p>Install all the software development packages needed including
<a href="#alsa">alsa</a>, <a href="#freetype">freetype</a>, <a href="#cups">cups</a>, and
<a href="#xrender">xrender</a>. See <a href="#SDBE">specific system packages</a>.</p></li>
<li><p><strong>Solaris</strong></p>

<p>Install all the software development packages needed including <a href="#studio">Studio
Compilers</a>, <a href="#freetype">freetype</a>, <a href="#cups">cups</a>, and
<a href="#xrender">xrender</a>. See <a href="#SDBE">specific system packages</a>.</p></li>
<li><p><strong>Windows</strong></p>

<ul>
<li>Install one of <a href="#cygwin">CYGWIN</a> or <a href="#msys">MinGW/MSYS</a></li>
<li>Install <a href="#vs2013">Visual Studio 2013</a></li>
</ul></li>
<li><p><strong>Mac OS X</strong></p>

<p>Install <a href="https://developer.apple.com/xcode/">XCode 6.3</a></p></li>
</ul>

<p><a name="linux"></a></p>

<h4>Linux</h4>

<p>With Linux, try and favor the system packages over building your own or getting
packages from other areas. Most Linux builds should be possible with the
system's available packages.</p>

<p>Note that some Linux systems have a habit of pre-populating your environment
variables for you, for example <code>JAVA_HOME</code> might get pre-defined for you to
refer to the JDK installed on your Linux system. You will need to unset
<code>JAVA_HOME</code>. It's a good idea to run <code>env</code> and verify the environment variables
you are getting from the default system settings make sense for building the
OpenJDK.</p>

<p><a name="solaris"></a></p>

<h4>Solaris</h4>

<p><a name="studio"></a></p>

<h5>Studio Compilers</h5>

<p>At a minimum, the <a href="http://www.oracle.com/
technetwork/server-storage/solarisstudio/downloads/index.htm">Studio 12 Update 4 Compilers</a> (containing
version 5.13 of the C and C++ compilers) is required, including specific
patches.</p>

<p>The Solaris Studio installation should contain at least these packages:</p>

<blockquote>
  <p><table border="1">
     <thead>
       <tr>
         <td><strong>Package</strong></td>
         <td><strong>Version</strong></td>
       </tr>
     </thead>
     <tbody>
       <tr>
         <td>developer/solarisstudio-124/backend</td>
         <td>12.4-1.0.6.0</td>
       </tr>
       <tr>
         <td>developer/solarisstudio-124/c++</td>
         <td>12.4-1.0.10.0</td>
       </tr>
       <tr>
         <td>developer/solarisstudio-124/cc</td>
         <td>12.4-1.0.4.0</td>
       </tr>
       <tr>
         <td>developer/solarisstudio-124/library/c++-libs</td>
         <td>12.4-1.0.10.0</td>
       </tr>
       <tr>
         <td>developer/solarisstudio-124/library/math-libs</td>
         <td>12.4-1.0.0.1</td>
       </tr>
       <tr>
         <td>developer/solarisstudio-124/library/studio-gccrt</td>
         <td>12.4-1.0.0.1</td>
       </tr>
       <tr>
         <td>developer/solarisstudio-124/studio-common</td>
         <td>12.4-1.0.0.1</td>
       </tr>
       <tr>
         <td>developer/solarisstudio-124/studio-ja</td>
         <td>12.4-1.0.0.1</td>
       </tr>
       <tr>
         <td>developer/solarisstudio-124/studio-legal</td>
         <td>12.4-1.0.0.1</td>
       </tr>
       <tr>
         <td>developer/solarisstudio-124/studio-zhCN</td>
         <td>12.4-1.0.0.1</td>
       </tr>
     </tbody>
   </table></p>
</blockquote>

<p>In particular backend 12.4-1.0.6.0 contains a critical patch for the sparc
version.</p>

<p>Place the <code>bin</code> directory in <code>PATH</code>.</p>

<p>The Oracle Solaris Studio Express compilers at: <a href="http://www.oracle.com/technetwork/server-storage/solarisstudio/
downloads/index-jsp-142582.html">Oracle Solaris Studio Express
Download site</a> are also an option, although these compilers
have not been extensively used yet.</p>

<p><a name="windows"></a></p>

<h4>Windows</h4>

<h5>Windows Unix Toolkit</h5>

<p>Building on Windows requires a Unix-like environment, notably a Unix-like
shell. There are several such environments available of which
<a href="http://www.cygwin.com/">Cygwin</a> and
<a href="http://www.mingw.org/wiki/MSYS">MinGW/MSYS</a> are currently supported for the
OpenJDK build. One of the differences of these systems from standard Windows
tools is the way they handle Windows path names, particularly path names which
contain spaces, backslashes as path separators and possibly drive letters.
Depending on the use case and the specifics of each environment these path
problems can be solved by a combination of quoting whole paths, translating
backslashes to forward slashes, escaping backslashes with additional
backslashes and translating the path names to their <a href="http://en.wikipedia.org/wiki/8.3_filename">"8.3"
version</a>.</p>

<p><a name="cygwin"></a></p>

<h6>CYGWIN</h6>

<p>CYGWIN is an open source, Linux-like environment which tries to emulate a
complete POSIX layer on Windows. It tries to be smart about path names and can
usually handle all kinds of paths if they are correctly quoted or escaped
although internally it maps drive letters <code>&lt;drive&gt;:</code> to a virtual directory
<code>/cygdrive/&lt;drive&gt;</code>.</p>

<p>You can always use the <code>cygpath</code> utility to map pathnames with spaces or the
backslash character into the <code>C:/</code> style of pathname (called 'mixed'), e.g.
<code>cygpath -s -m "&lt;path&gt;"</code>.</p>

<p>Note that the use of CYGWIN creates a unique problem with regards to setting
<a href="#path"><code>PATH</code></a>. Normally on Windows the <code>PATH</code> variable contains directories
separated with the ";" character (Solaris and Linux use ":"). With CYGWIN, it
uses ":", but that means that paths like "C:/path" cannot be placed in the
CYGWIN version of <code>PATH</code> and instead CYGWIN uses something like
<code>/cygdrive/c/path</code> which CYGWIN understands, but only CYGWIN understands.</p>

<p>The OpenJDK build requires CYGWIN version 1.7.16 or newer. Information about
CYGWIN can be obtained from the CYGWIN website at
<a href="http://www.cygwin.com">www.cygwin.com</a>.</p>

<p>By default CYGWIN doesn't install all the tools required for building the
OpenJDK. Along with the default installation, you need to install the following
tools.</p>

<blockquote>
  <p><table border="1">
     <thead>
       <tr>
         <td>Binary Name</td>
         <td>Category</td>
         <td>Package</td>
         <td>Description</td>
      </tr>
     </thead>
     <tbody>
       <tr>
         <td>ar.exe</td>
         <td>Devel</td>
         <td>binutils</td>
         <td>The GNU assembler, linker and binary utilities</td>
       </tr>
       <tr>
         <td>make.exe</td>
         <td>Devel</td>
         <td>make</td>
         <td>The GNU version of the 'make' utility built for CYGWIN</td>
       </tr>
       <tr>
         <td>m4.exe</td>
         <td>Interpreters</td>
         <td>m4</td>
         <td>GNU implementation of the traditional Unix macro processor</td>
       </tr>
       <tr>
         <td>cpio.exe</td>
         <td>Utils</td>
         <td>cpio</td>
         <td>A program to manage archives of files</td>
       </tr>
       <tr>
         <td>gawk.exe</td>
         <td>Utils</td>
         <td>awk</td>
         <td>Pattern-directed scanning and processing language</td>
       </tr>
       <tr>
         <td>file.exe</td>
         <td>Utils</td>
         <td>file</td>
         <td>Determines file type using 'magic' numbers</td>
       </tr>
       <tr>
         <td>zip.exe</td>
         <td>Archive</td>
         <td>zip</td>
         <td>Package and compress (archive) files</td>
       </tr>
       <tr>
         <td>unzip.exe</td>
         <td>Archive</td>
         <td>unzip</td>
         <td>Extract compressed files in a ZIP archive</td>
       </tr>
       <tr>
         <td>free.exe</td>
         <td>System</td>
         <td>procps</td>
         <td>Display amount of free and used memory in the system</td>
       </tr>
     </tbody>
   </table></p>
</blockquote>

<p>Note that the CYGWIN software can conflict with other non-CYGWIN software on
your Windows system. CYGWIN provides a <a href="http://cygwin.com/faq/
faq.using.html">FAQ</a> for known issues and problems, of particular interest is the
section on <a href="http://cygwin.com/faq/faq.using.html#faq.using.bloda">BLODA (applications that interfere with
CYGWIN)</a>.</p>

<p><a name="msys"></a></p>

<h6>MinGW/MSYS</h6>

<p>MinGW ("Minimalist GNU for Windows") is a collection of free Windows specific
header files and import libraries combined with GNU toolsets that allow one to
produce native Windows programs that do not rely on any 3rd-party C runtime
DLLs. MSYS is a supplement to MinGW which allows building applications and
programs which rely on traditional UNIX tools to be present. Among others this
includes tools like <code>bash</code> and <code>make</code>. See <a href="http://www.mingw.org/
wiki/MSYS">MinGW/MSYS</a> for more information.</p>

<p>Like Cygwin, MinGW/MSYS can handle different types of path formats. They are
internally converted to paths with forward slashes and drive letters
<code>&lt;drive&gt;:</code> replaced by a virtual directory <code>/&lt;drive&gt;</code>. Additionally, MSYS
automatically detects binaries compiled for the MSYS environment and feeds them
with the internal, Unix-style path names. If native Windows applications are
called from within MSYS programs their path arguments are automatically
converted back to Windows style path names with drive letters and backslashes
as path separators. This may cause problems for Windows applications which use
forward slashes as parameter separator (e.g. <code>cl /nologo /I</code>) because MSYS may
wrongly <a href="http://mingw.org/wiki/
Posix_path_conversion">replace such parameters by drive letters</a>.</p>

<p>In addition to the tools which will be installed by default, you have to
manually install the <code>msys-zip</code> and <code>msys-unzip</code> packages. This can be easily
done with the MinGW command line installer:</p>

<pre><code>  mingw-get.exe install msys-zip
  mingw-get.exe install msys-unzip
</code></pre>

<p><a name="vs2013"></a></p>

<h5>Visual Studio 2013 Compilers</h5>

<p>The 32-bit and 64-bit OpenJDK Windows build requires Microsoft Visual Studio
C++ 2013 (VS2013) Professional Edition or Express compiler. The compiler and
other tools are expected to reside in the location defined by the variable
<code>VS120COMNTOOLS</code> which is set by the Microsoft Visual Studio installer.</p>

<p>Only the C++ part of VS2013 is needed. Try to let the installation go to the
default install directory. Always reboot your system after installing VS2013.
The system environment variable VS120COMNTOOLS should be set in your
environment.</p>

<p>Make sure that TMP and TEMP are also set in the environment and refer to
Windows paths that exist, like <code>C:\temp</code>, not <code>/tmp</code>, not <code>/cygdrive/c/temp</code>,
and not <code>C:/temp</code>. <code>C:\temp</code> is just an example, it is assumed that this area
is private to the user, so by default after installs you should see a unique
user path in these variables.</p>

<p><a name="macosx"></a></p>

<h4>Mac OS X</h4>

<p>Make sure you get the right XCode version.</p>

<hr />

<p><a name="configure"></a></p>

<h3>Configure</h3>

<p>The basic invocation of the <code>configure</code> script looks like:</p>

<blockquote>
  <p><strong><code>bash ./configure [options]</code></strong></p>
</blockquote>

<p>This will create an output directory containing the "configuration" and setup
an area for the build result. This directory typically looks like:</p>

<blockquote>
  <p><strong><code>build/linux-x64-normal-server-release</code></strong></p>
</blockquote>

<p><code>configure</code> will try to figure out what system you are running on and where all
necessary build components are. If you have all prerequisites for building
installed, it should find everything. If it fails to detect any component
automatically, it will exit and inform you about the problem. When this
happens, read more below in <a href="#configureoptions">the <code>configure</code> options</a>.</p>

<p>Some examples:</p>

<blockquote>
  <p><strong>Windows 32bit build with freetype specified:</strong> <br />
 <code>bash ./configure --with-freetype=/cygdrive/c/freetype-i586 --with-target-
bits=32</code></p>

<p><strong>Debug 64bit Build:</strong> <br />
 <code>bash ./configure --enable-debug --with-target-bits=64</code></p>
</blockquote>

<p><a name="configureoptions"></a></p>

<h4>Configure Options</h4>

<p>Complete details on all the OpenJDK <code>configure</code> options can be seen with:</p>

<blockquote>
  <p><strong><code>bash ./configure --help=short</code></strong></p>
</blockquote>

<p>Use <code>-help</code> to see all the <code>configure</code> options available. You can generate any
number of different configurations, e.g. debug, release, 32, 64, etc.</p>

<p>Some of the more commonly used <code>configure</code> options are:</p>

<blockquote>
  <p><strong><code>--enable-debug</code></strong> <br />
 set the debug level to fastdebug (this is a shorthand for <code>--with-debug-
   level=fastdebug</code>)</p>
</blockquote>

<p><a name="alsa"></a></p>

<blockquote>
  <p><strong><code>--with-alsa=</code></strong><em>path</em> <br />
 select the location of the Advanced Linux Sound Architecture (ALSA)</p>

<p>Version 0.9.1 or newer of the ALSA files are required for building the
   OpenJDK on Linux. These Linux files are usually available from an "alsa" of
   "libasound" development package, and it's highly recommended that you try
   and use the package provided by the particular version of Linux that you are
   using.</p>

<p><strong><code>--with-boot-jdk=</code></strong><em>path</em> <br />
 select the <a href="#bootjdk">Bootstrap JDK</a></p>

<p><strong><code>--with-boot-jdk-jvmargs=</code></strong>"<em>args</em>" <br />
 provide the JVM options to be used to run the <a href="#bootjdk">Bootstrap JDK</a></p>

<p><strong><code>--with-cacerts=</code></strong><em>path</em> <br />
 select the path to the cacerts file.</p>

<p>See <a href="http://en.wikipedia.org/wiki/
   Certificate_Authority">Certificate Authority on Wikipedia</a> for a better understanding of the Certificate
   Authority (CA). A certificates file named "cacerts" represents a system-wide
   keystore with CA certificates. In JDK and JRE binary bundles, the "cacerts"
   file contains root CA certificates from several public CAs (e.g., VeriSign,
   Thawte, and Baltimore). The source contain a cacerts file without CA root
   certificates. Formal JDK builders will need to secure permission from each
   public CA and include the certificates into their own custom cacerts file.
   Failure to provide a populated cacerts file will result in verification
   errors of a certificate chain during runtime. By default an empty cacerts
   file is provided and that should be fine for most JDK developers.</p>
</blockquote>

<p><a name="cups"></a></p>

<blockquote>
  <p><strong><code>--with-cups=</code></strong><em>path</em> <br />
 select the CUPS install location</p>

<p>The Common UNIX Printing System (CUPS) Headers are required for building the
   OpenJDK on Solaris and Linux. The Solaris header files can be obtained by
   installing the package <strong>print/cups</strong>.</p>

<p>The CUPS header files can always be downloaded from
   <a href="http://www.cups.org">www.cups.org</a>.</p>

<p><strong><code>--with-cups-include=</code></strong><em>path</em> <br />
 select the CUPS include directory location</p>

<p><strong><code>--with-debug-level=</code></strong><em>level</em> <br />
 select the debug information level of release, fastdebug, or slowdebug</p>

<p><strong><code>--with-dev-kit=</code></strong><em>path</em> <br />
 select location of the compiler install or developer install location</p>
</blockquote>

<p><a name="freetype"></a></p>

<blockquote>
  <p><strong><code>--with-freetype=</code></strong><em>path</em> <br />
 select the freetype files to use.</p>

<p>Expecting the freetype libraries under <code>lib/</code> and the headers under
   <code>include/</code>.</p>

<p>Version 2.3 or newer of FreeType is required. On Unix systems required files
   can be available as part of your distribution (while you still may need to
   upgrade them). Note that you need development version of package that
   includes both the FreeType library and header files.</p>

<p>You can always download latest FreeType version from the <a href="http://www.freetype.org">FreeType
   website</a>. Building the freetype 2 libraries from
   scratch is also possible, however on Windows refer to the <a href="http://freetype.freedesktop.org/wiki/FreeType_DLL">Windows FreeType
   DLL build instructions</a>.</p>

<p>Note that by default FreeType is built with byte code hinting support
   disabled due to licensing restrictions. In this case, text appearance and
   metrics are expected to differ from Sun's official JDK build. See the
   <a href="http://freetype.sourceforge.net/freetype2">SourceForge FreeType2 Home Page</a>
   for more information.</p>

<p><strong><code>--with-import-hotspot=</code></strong><em>path</em> <br />
 select the location to find hotspot binaries from a previous build to avoid
   building hotspot</p>

<p><strong><code>--with-target-bits=</code></strong><em>arg</em> <br />
 select 32 or 64 bit build</p>

<p><strong><code>--with-jvm-variants=</code></strong><em>variants</em> <br />
 select the JVM variants to build from, comma separated list that can
   include: server, client, kernel, zero and zeroshark</p>

<p><strong><code>--with-memory-size=</code></strong><em>size</em> <br />
 select the RAM size that GNU make will think this system has</p>

<p><strong><code>--with-msvcr-dll=</code></strong><em>path</em> <br />
 select the <code>msvcr100.dll</code> file to include in the Windows builds (C/C++
   runtime library for Visual Studio).</p>

<p>This is usually picked up automatically from the redist directories of
   Visual Studio 2013.</p>

<p><strong><code>--with-num-cores=</code></strong><em>cores</em> <br />
 select the number of cores to use (processor count or CPU count)</p>
</blockquote>

<p><a name="xrender"></a></p>

<blockquote>
  <p><strong><code>--with-x=</code></strong><em>path</em> <br />
 select the location of the X11 and xrender files.</p>

<p>The XRender Extension Headers are required for building the OpenJDK on
   Solaris and Linux. The Linux header files are usually available from a
   "Xrender" development package, it's recommended that you try and use the
   package provided by the particular distribution of Linux that you are using.
   The Solaris XRender header files is included with the other X11 header files
   in the package <strong>SFWxwinc</strong> on new enough versions of Solaris and will be
   installed in <code>/usr/X11/include/X11/extensions/Xrender.h</code> or
   <code>/usr/openwin/share/include/X11/extensions/Xrender.h</code></p>
</blockquote>

<hr />

<p><a name="make"></a></p>

<h3>Make</h3>

<p>The basic invocation of the <code>make</code> utility looks like:</p>

<blockquote>
  <p><strong><code>make all</code></strong></p>
</blockquote>

<p>This will start the build to the output directory containing the
"configuration" that was created by the <code>configure</code> script. Run <code>make help</code> for
more information on the available targets.</p>

<p>There are some of the make targets that are of general interest:</p>

<blockquote>
  <p><em>empty</em> <br />
 build everything but no images</p>

<p><strong><code>all</code></strong> <br />
 build everything including images</p>

<p><strong><code>all-conf</code></strong> <br />
 build all configurations</p>

<p><strong><code>images</code></strong> <br />
 create complete j2sdk and j2re images</p>

<p><strong><code>install</code></strong> <br />
 install the generated images locally, typically in <code>/usr/local</code></p>

<p><strong><code>clean</code></strong> <br />
 remove all files generated by make, but not those generated by <code>configure</code></p>

<p><strong><code>dist-clean</code></strong> <br />
 remove all files generated by both and <code>configure</code> (basically killing the
   configuration)</p>

<p><strong><code>help</code></strong> <br />
 give some help on using <code>make</code>, including some interesting make targets</p>
</blockquote>

<hr />

<p><a name="testing"></a></p>

<h2>Testing</h2>

<p>When the build is completed, you should see the generated binaries and
associated files in the <code>j2sdk-image</code> directory in the output directory. In
particular, the <code>build/*/images/j2sdk-image/bin</code> directory should contain
executables for the OpenJDK tools and utilities for that configuration. The
testing tool <code>jtreg</code> will be needed and can be found at: <a href="http://openjdk.java.net/jtreg/">the jtreg
site</a>. The provided regression tests in the
repositories can be run with the command:</p>

<blockquote>
  <p><strong><code>cd test &amp;&amp; make PRODUCT_HOME=`pwd`/../build/*/images/j2sdk-image all</code></strong></p>
</blockquote>

<hr />

<p><a name="hints"></a></p>

<h2>Appendix A: Hints and Tips</h2>

<p><a name="faq"></a></p>

<h3>FAQ</h3>

<p><strong>Q:</strong> The <code>generated-configure.sh</code> file looks horrible! How are you going to
edit it? <br />
<strong>A:</strong> The <code>generated-configure.sh</code> file is generated (think "compiled") by the
autoconf tools. The source code is in <code>configure.ac</code> and various .m4 files in
common/autoconf, which are much more readable.</p>

<p><strong>Q:</strong> Why is the <code>generated-configure.sh</code> file checked in, if it is 
generated? <br />
<strong>A:</strong> If it was not generated, every user would need to have the autoconf
tools installed, and re-generate the <code>configure</code> file as the first step. Our
goal is to minimize the work needed to be done by the user to start building
OpenJDK, and to minimize the number of external dependencies required.</p>

<p><strong>Q:</strong> Do you require a specific version of autoconf for regenerating
<code>generated-configure.sh</code>? <br />
<strong>A:</strong> Yes, version 2.69 is required and should be easy enough to aquire on all
supported operating systems. The reason for this is to avoid large spurious
changes in <code>generated-configure.sh</code>.</p>

<p><strong>Q:</strong> How do you regenerate <code>generated-configure.sh</code> after making changes to
the input files? <br />
<strong>A:</strong> Regnerating <code>generated-configure.sh</code> should always be done using the
script <code>common/autoconf/autogen.sh</code> to ensure that the correct files get
updated. This script should also be run after mercurial tries to merge
<code>generated-configure.sh</code> as a merge of the generated file is not guaranteed to
be correct.</p>

<p><strong>Q:</strong> What are the files in <code>common/makefiles/support/*</code> for? They look like
gibberish. <br />
<strong>A:</strong> They are a somewhat ugly hack to compensate for command line length
limitations on certain platforms (Windows, Solaris). Due to a combination of
limitations in make and the shell, command lines containing too many files will
not work properly. These helper files are part of an elaborate hack that will
compress the command line in the makefile and then uncompress it safely. We're
not proud of it, but it does fix the problem. If you have any better
suggestions, we're all ears! :-)</p>

<p><strong>Q:</strong> I want to see the output of the commands that make runs, like in the old
build. How do I do that? <br />
<strong>A:</strong> You specify the <code>LOG</code> variable to make. There are several log levels:</p>

<ul>
<li><strong><code>warn</code></strong> -- Default and very quiet.</li>
<li><strong><code>info</code></strong> -- Shows more progress information than warn.</li>
<li><strong><code>debug</code></strong> -- Echos all command lines and prints all macro calls for
compilation definitions.</li>
<li><strong><code>trace</code></strong> -- Echos all $(shell) command lines as well.</li>
</ul>

<p><strong>Q:</strong> When do I have to re-run <code>configure</code>? <br />
<strong>A:</strong> Normally you will run <code>configure</code> only once for creating a
configuration. You need to re-run configuration only if you want to change any
configuration options, or if you pull down changes to the <code>configure</code> script.</p>

<p><strong>Q:</strong> I have added a new source file. Do I need to modify the makefiles? <br />
<strong>A:</strong> Normally, no. If you want to create e.g. a new native library, you will
need to modify the makefiles. But for normal file additions or removals, no
changes are needed. There are certan exceptions for some native libraries where
the source files are spread over many directories which also contain sources
for other libraries. In these cases it was simply easier to create include
lists rather than excludes.</p>

<p><strong>Q:</strong> When I run <code>configure --help</code>, I see many strange options, like
<code>--dvidir</code>. What is this? <br />
<strong>A:</strong> Configure provides a slew of options by default, to all projects that
use autoconf. Most of them are not used in OpenJDK, so you can safely ignore
them. To list only OpenJDK specific features, use <code>configure --help=short</code>
instead.</p>

<p><strong>Q:</strong> <code>configure</code> provides OpenJDK-specific features such as <code>--with-
builddeps-server</code> that are not described in this document. What about those? <br />
<strong>A:</strong> Try them out if you like! But be aware that most of these are
experimental features. Many of them don't do anything at all at the moment; the
option is just a placeholder. Others depend on pieces of code or infrastructure
that is currently not ready for prime time.</p>

<p><strong>Q:</strong> How will you make sure you don't break anything? <br />
<strong>A:</strong> We have a script that compares the result of the new build system with
the result of the old. For most part, we aim for (and achieve) byte-by-byte
identical output. There are however technical issues with e.g. native binaries,
which might differ in a byte-by-byte comparison, even when building twice with
the old build system. For these, we compare relevant aspects (e.g. the symbol
table and file size). Note that we still don't have 100% equivalence, but we're
close.</p>

<p><strong>Q:</strong> I noticed this thing X in the build that looks very broken by design.
Why don't you fix it? <br />
<strong>A:</strong> Our goal is to produce a build output that is as close as technically
possible to the old build output. If things were weird in the old build, they
will be weird in the new build. Often, things were weird before due to
obscurity, but in the new build system the weird stuff comes up to the surface.
The plan is to attack these things at a later stage, after the new build system
is established.</p>

<p><strong>Q:</strong> The code in the new build system is not that well-structured. Will you
fix this? <br />
<strong>A:</strong> Yes! The new build system has grown bit by bit as we converted the old
system. When all of the old build system is converted, we can take a step back
and clean up the structure of the new build system. Some of this we plan to do
before replacing the old build system and some will need to wait until after.</p>

<p><strong>Q:</strong> Is anything able to use the results of the new build's default make
target? <br />
<strong>A:</strong> Yes, this is the minimal (or roughly minimal) set of compiled output
needed for a developer to actually execute the newly built JDK. The idea is
that in an incremental development fashion, when doing a normal make, you
should only spend time recompiling what's changed (making it purely
incremental) and only do the work that's needed to actually run and test your
code. The packaging stuff that is part of the <code>images</code> target is not needed for
a normal developer who wants to test his new code. Even if it's quite fast,
it's still unnecessary. We're targeting sub-second incremental rebuilds! ;-)
(Or, well, at least single-digit seconds...)</p>

<p><strong>Q:</strong> I usually set a specific environment variable when building, but I can't
find the equivalent in the new build. What should I do? <br />
<strong>A:</strong> It might very well be that we have neglected to add support for an
option that was actually used from outside the build system. Email us and we
will add support for it!</p>

<p><a name="performance"></a></p>

<h3>Build Performance Tips</h3>

<p>Building OpenJDK requires a lot of horsepower. Some of the build tools can be
adjusted to utilize more or less of resources such as parallel threads and
memory. The <code>configure</code> script analyzes your system and selects reasonable
values for such options based on your hardware. If you encounter resource
problems, such as out of memory conditions, you can modify the detected values
with:</p>

<ul>
<li><strong><code>--with-num-cores</code></strong> -- number of cores in the build system, e.g.
<code>--with-num-cores=8</code></li>
<li><strong><code>--with-memory-size</code></strong> -- memory (in MB) available in the build system,
e.g. <code>--with-memory-size=1024</code></li>
</ul>

<p>It might also be necessary to specify the JVM arguments passed to the Bootstrap
JDK, using e.g. <code>--with-boot-jdk-jvmargs="-Xmx8G -enableassertions"</code>. Doing
this will override the default JVM arguments passed to the Bootstrap JDK.</p>

<p>One of the top goals of the new build system is to improve the build
performance and decrease the time needed to build. This will soon also apply to
the java compilation when the Smart Javac wrapper is fully supported.</p>

<p>At the end of a successful execution of <code>configure</code>, you will get a performance
summary, indicating how well the build will perform. Here you will also get
performance hints. If you want to build fast, pay attention to those!</p>

<h4>Building with ccache</h4>

<p>The OpenJDK build supports building with ccache when using gcc or clang. Using
ccache can radically speed up compilation of native code if you often rebuild
the same sources. Your milage may vary however so we recommend evaluating it
for yourself. To enable it, make sure it's on the path and configure with
<code>--enable-ccache</code>.</p>

<h4>Building on local disk</h4>

<p>If you are using network shares, e.g. via NFS, for your source code, make sure
the build directory is situated on local disk. The performance penalty is
extremely high for building on a network share, close to unusable.</p>

<h4>Building only one JVM</h4>

<p>The old build builds multiple JVMs on 32-bit systems (client and server; and on
Windows kernel as well). In the new build we have changed this default to only
build server when it's available. This improves build times for those not
interested in multiple JVMs. To mimic the old behavior on platforms that
support it, use <code>--with-jvm-variants=client,server</code>.</p>

<h4>Selecting the number of cores to build on</h4>

<p>By default, <code>configure</code> will analyze your machine and run the make process in
parallel with as many threads as you have cores. This behavior can be
overridden, either "permanently" (on a <code>configure</code> basis) using
<code>--with-num-cores=N</code> or for a single build only (on a make basis), using
<code>make JOBS=N</code>.</p>

<p>If you want to make a slower build just this time, to save some CPU power for
other processes, you can run e.g. <code>make JOBS=2</code>. This will force the makefiles
to only run 2 parallel processes, or even <code>make JOBS=1</code> which will disable
parallelism.</p>

<p>If you want to have it the other way round, namely having slow builds default
and override with fast if you're impatient, you should call <code>configure</code> with
<code>--with-num-cores=2</code>, making 2 the default. If you want to run with more cores,
run <code>make JOBS=8</code></p>

<p><a name="troubleshooting"></a></p>

<h3>Troubleshooting</h3>

<h4>Solving build problems</h4>

<p>If the build fails (and it's not due to a compilation error in a source file
you've changed), the first thing you should do is to re-run the build with more
verbosity. Do this by adding <code>LOG=debug</code> to your make command line.</p>

<p>The build log (with both stdout and stderr intermingled, basically the same as
you see on your console) can be found as <code>build.log</code> in your build directory.</p>

<p>You can ask for help on build problems with the new build system on either the
<a href="http://mail.openjdk.java.net/mailman/listinfo/build-dev">build-dev</a> or the
<a href="http://mail.openjdk.java.net/mailman/listinfo/build-infra-dev">build-infra-dev</a>
mailing lists. Please include the relevant parts of the build log.</p>

<p>A build can fail for any number of reasons. Most failures are a result of
trying to build in an environment in which all the pre-build requirements have
not been met. The first step in troubleshooting a build failure is to recheck
that you have satisfied all the pre-build requirements for your platform.
Scanning the <code>configure</code> log is a good first step, making sure that what it
found makes sense for your system. Look for strange error messages or any
difficulties that <code>configure</code> had in finding things.</p>

<p>Some of the more common problems with builds are briefly described below, with
suggestions for remedies.</p>

<ul>
<li><p><strong>Corrupted Bundles on Windows:</strong> <br />
Some virus scanning software has been known to corrupt the downloading of
zip bundles. It may be necessary to disable the 'on access' or 'real time'
virus scanning features to prevent this corruption. This type of 'real time'
virus scanning can also slow down the build process significantly.
Temporarily disabling the feature, or excluding the build output directory
may be necessary to get correct and faster builds.</p></li>
<li><p><strong>Slow Builds:</strong> <br />
If your build machine seems to be overloaded from too many simultaneous C++
compiles, try setting the <code>JOBS=1</code> on the <code>make</code> command line. Then try
increasing the count slowly to an acceptable level for your system. Also:</p>

<p>Creating the javadocs can be very slow, if you are running javadoc, consider
skipping that step.</p>

<p>Faster CPUs, more RAM, and a faster DISK usually helps. The VM build tends
to be CPU intensive (many C++ compiles), and the rest of the JDK will often
be disk intensive.</p>

<p>Faster compiles are possible using a tool called
<a href="http://ccache.samba.org/">ccache</a>.</p></li>
<li><p><strong>File time issues:</strong> <br />
If you see warnings that refer to file time stamps, e.g.</p>

<blockquote>
  <p><em>Warning message:</em> <code>File 'xxx' has modification time in the future.</code> <br />
<em>Warning message:</em> <code>Clock skew detected. Your build may be incomplete.</code></p>
</blockquote>

<p>These warnings can occur when the clock on the build machine is out of sync
with the timestamps on the source files. Other errors, apparently unrelated
but in fact caused by the clock skew, can occur along with the clock skew
warnings. These secondary errors may tend to obscure the fact that the true
root cause of the problem is an out-of-sync clock.</p>

<p>If you see these warnings, reset the clock on the build machine, run
"<code>gmake clobber</code>" or delete the directory containing the build output, and
restart the build from the beginning.</p></li>
<li><p><strong>Error message: <code>Trouble writing out table to disk</code></strong> <br />
Increase the amount of swap space on your build machine. This could be
caused by overloading the system and it may be necessary to use:</p>

<blockquote>
  <p><code>make JOBS=1</code></p>
</blockquote>

<p>to reduce the load on the system.</p></li>
<li><p><strong>Error Message: <code>libstdc++ not found</code>:</strong> <br />
This is caused by a missing libstdc++.a library. This is installed as part
of a specific package (e.g. libstdc++.so.devel.386). By default some 64-bit
Linux versions (e.g. Fedora) only install the 64-bit version of the
libstdc++ package. Various parts of the JDK build require a static link of
the C++ runtime libraries to allow for maximum portability of the built
images.</p></li>
<li><p><strong>Linux Error Message: <code>cannot restore segment prot after reloc</code></strong> <br />
This is probably an issue with SELinux (See <a href="http://en.wikipedia.org/wiki/SELinux">SELinux on
Wikipedia</a>). Parts of the VM is built
without the <code>-fPIC</code> for performance reasons.</p>

<p>To completely disable SELinux:</p>

<ol>
<li><code>$ su root</code></li>
<li><code># system-config-securitylevel</code></li>
<li><code>In the window that appears, select the SELinux tab</code></li>
<li><code>Disable SELinux</code></li>
</ol>

<p>Alternatively, instead of completely disabling it you could disable just
this one check.</p>

<ol>
<li>Select System->Administration->SELinux Management</li>
<li>In the SELinux Management Tool which appears, select "Boolean" from the
menu on the left</li>
<li>Expand the "Memory Protection" group</li>
<li>Check the first item, labeled "Allow all unconfined executables to use
libraries requiring text relocation ..."</li>
</ol></li>
<li><p><strong>Windows Error Messages:</strong> <br />
<code>*** fatal error - couldn't allocate heap, ...</code> <br />
<code>rm fails with "Directory not empty"</code> <br />
<code>unzip fails with "cannot create ... Permission denied"</code> <br />
<code>unzip fails with "cannot create ... Error 50"</code></p>

<p>The CYGWIN software can conflict with other non-CYGWIN software. See the
CYGWIN FAQ section on <a href="http://cygwin.com/faq/faq.using.html#faq.using.bloda">BLODA (applications that interfere with
CYGWIN)</a>.</p></li>
<li><p><strong>Windows Error Message: <code>spawn failed</code></strong> <br />
Try rebooting the system, or there could be some kind of issue with the disk
or disk partition being used. Sometimes it comes with a "Permission Denied"
message.</p></li>
</ul>

<hr />

<p><a name="gmake"></a></p>

<h2>Appendix B: GNU make</h2>

<p>The Makefiles in the OpenJDK are only valid when used with the GNU version of
the utility command <code>make</code> (usually called <code>gmake</code> on Solaris). A few notes
about using GNU make:</p>

<ul>
<li>You need GNU make version 3.81 or newer. On Windows 4.0 or newer is
recommended. If the GNU make utility on your systems is not of a suitable
version, see "<a href="#buildgmake">Building GNU make</a>".</li>
<li>Place the location of the GNU make binary in the <code>PATH</code>.</li>
<li><strong>Solaris:</strong> Do NOT use <code>/usr/bin/make</code> on Solaris. If your Solaris system
has the software from the Solaris Developer Companion CD installed, you
should try and use <code>/usr/bin/gmake</code> or <code>/usr/gnu/bin/make</code>.</li>
<li><strong>Windows:</strong> Make sure you start your build inside a bash shell.</li>
<li><strong>Mac OS X:</strong> The XCode "command line tools" must be installed on your Mac.</li>
</ul>

<p>Information on GNU make, and access to ftp download sites, are available on the
<a href="http://www.gnu.org/software/make/make.html">GNU make web site </a>. The latest
source to GNU make is available at
<a href="http://ftp.gnu.org/pub/gnu/make/">ftp.gnu.org/pub/gnu/make/</a>.</p>

<p><a name="buildgmake"></a></p>

<h3>Building GNU make</h3>

<p>First step is to get the GNU make 3.81 or newer source from
<a href="http://ftp.gnu.org/pub/gnu/make/">ftp.gnu.org/pub/gnu/make/</a>. Building is a
little different depending on the OS but is basically done with:</p>

<pre><code>  bash ./configure
  make
</code></pre>

<hr />

<p><a name="buildenvironments"></a></p>

<h2>Appendix C: Build Environments</h2>

<h3>Minimum Build Environments</h3>

<p>This file often describes specific requirements for what we call the "minimum
build environments" (MBE) for this specific release of the JDK. What is listed
below is what the Oracle Release Engineering Team will use to build the Oracle
JDK product. Building with the MBE will hopefully generate the most compatible
bits that install on, and run correctly on, the most variations of the same
base OS and hardware architecture. In some cases, these represent what is often
called the least common denominator, but each Operating System has different
aspects to it.</p>

<p>In all cases, the Bootstrap JDK version minimum is critical, we cannot
guarantee builds will work with older Bootstrap JDK's. Also in all cases, more
RAM and more processors is better, the minimums listed below are simply
recommendations.</p>

<p>With Solaris and Mac OS X, the version listed below is the oldest release we
can guarantee builds and works, and the specific version of the compilers used
could be critical.</p>

<p>With Windows the critical aspect is the Visual Studio compiler used, which due
to it's runtime, generally dictates what Windows systems can do the builds and
where the resulting bits can be used.</p>

<p><strong>NOTE: We expect a change here off these older Windows OS releases and to a
'less older' one, probably Windows 2008R2 X64.</strong></p>

<p>With Linux, it was just a matter of picking a stable distribution that is a
good representative for Linux in general.</p>

<p>It is understood that most developers will NOT be using these specific
versions, and in fact creating these specific versions may be difficult due to
the age of some of this software. It is expected that developers are more often
using the more recent releases and distributions of these operating systems.</p>

<p>Compilation problems with newer or different C/C++ compilers is a common
problem. Similarly, compilation problems related to changes to the
<code>/usr/include</code> or system header files is also a common problem with older,
newer, or unreleased OS versions. Please report these types of problems as bugs
so that they can be dealt with accordingly.</p>

<blockquote>
  <p><table border="1">
     <thead>
       <tr>
         <th>Base OS and Architecture</th>
         <th>OS</th>
         <th>C/C++ Compiler</th>
         <th>Bootstrap JDK</th>
         <th>Processors</th>
         <th>RAM Minimum</th>
         <th>DISK Needs</th>
       </tr>
     </thead>
     <tbody>
       <tr>
         <td>Linux X86 (32-bit) and X64 (64-bit)</td>
         <td>Oracle Enterprise Linux 6.4</td>
         <td>gcc 4.9.2 </td>
         <td>JDK 8</td>
         <td>2 or more</td>
         <td>1 GB</td>
         <td>6 GB</td>
       </tr>
       <tr>
         <td>Solaris SPARCV9 (64-bit)</td>
         <td>Solaris 11 Update 1</td>
         <td>Studio 12 Update 4 + patches</td>
         <td>JDK 8</td>
         <td>4 or more</td>
         <td>4 GB</td>
         <td>8 GB</td>
       </tr>
       <tr>
         <td>Solaris X64 (64-bit)</td>
         <td>Solaris 11 Update 1</td>
         <td>Studio 12 Update 4 + patches</td>
         <td>JDK 8</td>
         <td>4 or more</td>
         <td>4 GB</td>
         <td>8 GB</td>
       </tr>
       <tr>
         <td>Windows X86 (32-bit)</td>
         <td>Windows Server 2012 R2 x64</td>
         <td>Microsoft Visual Studio C++ 2013 Professional Edition</td>
         <td>JDK 8</td>
         <td>2 or more</td>
         <td>2 GB</td>
         <td>6 GB</td>
       </tr>
       <tr>
         <td>Windows X64 (64-bit)</td>
         <td>Windows Server 2012 R2 x64</td>
         <td>Microsoft Visual Studio C++ 2013 Professional Edition</td>
         <td>JDK 8</td>
         <td>2 or more</td>
         <td>2 GB</td>
         <td>6 GB</td>
       </tr>
       <tr>
         <td>Mac OS X X64 (64-bit)</td>
         <td>Mac OS X 10.9 "Mavericks"</td>
         <td>Xcode 6.3 or newer</td>
         <td>JDK 8</td>
         <td>2 or more</td>
         <td>4 GB</td>
         <td>6 GB</td>
       </tr>
     </tbody>
   </table></p>
</blockquote>

<hr />

<p><a name="SDBE"></a></p>

<h3>Specific Developer Build Environments</h3>

<p>We won't be listing all the possible environments, but we will try to provide
what information we have available to us.</p>

<p><strong>NOTE: The community can help out by updating this part of the document.</strong></p>

<h4>Fedora</h4>

<p>After installing the latest <a href="http://fedoraproject.org">Fedora</a> you need to
install several build dependencies. The simplest way to do it is to execute the
following commands as user <code>root</code>:</p>

<pre><code>  yum-builddep java-1.7.0-openjdk
  yum install gcc gcc-c++
</code></pre>

<p>In addition, it's necessary to set a few environment variables for the build:</p>

<pre><code>  export LANG=C
  export PATH="/usr/lib/jvm/java-openjdk/bin:${PATH}"
</code></pre>

<h4>CentOS 5.5</h4>

<p>After installing <a href="http://www.centos.org/">CentOS 5.5</a> you need to make sure you
have the following Development bundles installed:</p>

<ul>
<li>Development Libraries</li>
<li>Development Tools</li>
<li>Java Development</li>
<li>X Software Development (Including XFree86-devel)</li>
</ul>

<p>Plus the following packages:</p>

<ul>
<li>cups devel: Cups Development Package</li>
<li>alsa devel: Alsa Development Package</li>
<li>Xi devel: libXi.so Development Package</li>
</ul>

<p>The freetype 2.3 packages don't seem to be available, but the freetype 2.3
sources can be downloaded, built, and installed easily enough from <a href="http://downloads.sourceforge.net/freetype">the
freetype site</a>. Build and install
with something like:</p>

<pre><code>  bash ./configure
  make
  sudo -u root make install
</code></pre>

<p>Mercurial packages could not be found easily, but a Google search should find
ones, and they usually include Python if it's needed.</p>

<h4>Debian 5.0 (Lenny)</h4>

<p>After installing <a href="http://debian.org">Debian</a> 5 you need to install several
build dependencies. The simplest way to install the build dependencies is to
execute the following commands as user <code>root</code>:</p>

<pre><code>  aptitude build-dep openjdk-7
  aptitude install openjdk-7-jdk libmotif-dev
</code></pre>

<p>In addition, it's necessary to set a few environment variables for the build:</p>

<pre><code>  export LANG=C
  export PATH="/usr/lib/jvm/java-7-openjdk/bin:${PATH}"
</code></pre>

<h4>Ubuntu 12.04</h4>

<p>After installing <a href="http://ubuntu.org">Ubuntu</a> 12.04 you need to install several
build dependencies. The simplest way to do it is to execute the following
commands:</p>

<pre><code>  sudo aptitude build-dep openjdk-7
  sudo aptitude install openjdk-7-jdk
</code></pre>

<p>In addition, it's necessary to set a few environment variables for the build:</p>

<pre><code>  export LANG=C
  export PATH="/usr/lib/jvm/java-7-openjdk/bin:${PATH}"
</code></pre>

<h4>OpenSUSE 11.1</h4>

<p>After installing <a href="http://opensuse.org">OpenSUSE</a> 11.1 you need to install
several build dependencies. The simplest way to install the build dependencies
is to execute the following commands:</p>

<pre><code>  sudo zypper source-install -d java-1_7_0-openjdk
  sudo zypper install make
</code></pre>

<p>In addition, it is necessary to set a few environment variables for the build:</p>

<pre><code>  export LANG=C
  export PATH="/usr/lib/jvm/java-1.7.0-openjdk/bin:$[PATH}"
</code></pre>

<p>Finally, you need to unset the <code>JAVA_HOME</code> environment variable:</p>

<pre><code>  export -n JAVA_HOME`
</code></pre>

<h4>Mandriva Linux One 2009 Spring</h4>

<p>After installing <a href="http://mandriva.org">Mandriva</a> Linux One 2009 Spring you need
to install several build dependencies. The simplest way to install the build
dependencies is to execute the following commands as user <code>root</code>:</p>

<pre><code>  urpmi java-1.7.0-openjdk-devel make gcc gcc-c++ freetype-devel zip unzip
    libcups2-devel libxrender1-devel libalsa2-devel libstc++-static-devel
    libxtst6-devel libxi-devel
</code></pre>

<p>In addition, it is necessary to set a few environment variables for the build:</p>

<pre><code>  export LANG=C
  export PATH="/usr/lib/jvm/java-1.7.0-openjdk/bin:${PATH}"
</code></pre>

<h4>OpenSolaris 2009.06</h4>

<p>After installing <a href="http://opensolaris.org">OpenSolaris</a> 2009.06 you need to
install several build dependencies. The simplest way to install the build
dependencies is to execute the following commands:</p>

<pre><code>  pfexec pkg install SUNWgmake SUNWj7dev sunstudioexpress SUNWcups SUNWzip
    SUNWunzip SUNWxwhl SUNWxorg-headers SUNWaudh SUNWfreetype2
</code></pre>

<p>In addition, it is necessary to set a few environment variables for the build:</p>

<pre><code>  export LANG=C
  export PATH="/opt/SunStudioExpress/bin:${PATH}"
</code></pre>

<hr />

<p>End of the OpenJDK build README document.</p>

<p>Please come again!</p>



