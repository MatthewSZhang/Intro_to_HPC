# Using the High Performance Computing (HPC) systems at Sheffield

## Presentation

The presentation given during the session is at https://mikecroucher.github.io/Intro_to_HPC/#/

## Description of Sheffield's HPC Systems

The University of Sheffield has two HPC systems:-

* [Sharc](http://docs.hpc.shef.ac.uk/en/latest/sharc/) Sheffield's newest system. It contains about 2000 CPU cores all of which are latest generation.
* [Iceberg](http://docs.hpc.shef.ac.uk/en/latest/iceberg/) Iceberg is Sheffield's old system. It contains 3440 CPU cores but many of them are very old and slow.

The two systems are broadly similar but have small differences in the way you load applications using the module system.

**We recommend that you use Sharc as much as possible.**

## Connecting to the HPC systems from Windows

### Exercise 1: Install MobaXterm

To use the HPC from a Windows machine, you need a way to connect - we recommend you install  `mobaXterm`.
This is available from [http://mobaxterm.mobatek.net](http://mobaxterm.mobatek.net).
On a University machine, you need to install the `portable` version (highlighted in the image below):

<img src="images/mobaXterm_download.png" />

The download is a zip file that contains three other files. You should **Extract these files**, for example to your desktop, before you use them.
Do not run MobaXterm directly from the zip file.

`mobaXterm` also contains `mobaTextEditor` which you can use to write your programs.

### Exercise 2: Log in to Sharc

You can connect to `Sharc` using `mobaXterm` as shown in the screenshot below.
The `Remote Host` field should contain `sharc.sheffield.ac.uk`:

<img src="images/mobaxterm_sharc.png" />

If your log-in is successful, you should see something like the screen below.

<img src="images/sharc_login.png" />

At this point, you are on the `log in` or `Master node` of Sharc. There isn't much compute power here and many people use it simultaneously. As such, we should get onto a compute node as fast as possible.

<img src="images/log-in.gif" />

### Exercise 3: Start an interactive session on a compute node.

Since Sharc is a shared system, used by 100s of users, we need to request some resources from the `scheduler` using the command `qrshx`. We need to tell the system how much memory we want to use.

For example, to request 8 Gigabytes (8G) of memory, we would enter

`qrshx -l rmem=8G`

**Note: the `l` is a small letter `L` not the number `1`**

If this command is successful, you should see the prompt change from `sharc-login1` or  `sharc-login-2` to `sharc-nodeXXX` where XXX will be replaced with the number of the node you have been assigned.

<img src="images/worker_shell.png" />

You are now on a compute node and have access to your own CPU core and 8 Gigabytes of RAM.

Now would be a good time to learn some Linux commands using our [Mini Terminal Tutorial](terminal_tutorial.md)

### Exercise 4: Download Scala/Spark 'Hello World' (one we made earlier)

To run a `scala` program on a Linux machine, it will need to be compiled using the [Scala build tool](http://www.scala-sbt.org/). This requires a very strict directory structure *and* a `.sbt` file specifying dependencies. We illustrate this on the `helloWorld` example.

On the compute node, download a prepared `Hello World` application from GitHub with the command

`git clone https://github.com/mikecroucher/scala-spark-HelloWorld`

Enter the directory containing the code with the command

`cd scala-spark-HelloWorld/
`

List the files in this directory with the Linux command `ls`

```
ls
```

Should give the output

```
project.sbt  README.md  src
```

Take a look at the contents of `project.sbt`, which defines our project, with the Linux command `more`

```
more project.sbt
```

Should give the output

```
name := "hello"
version := "1.0"
scalaVersion := "2.11.8"
libraryDependencies ++= Seq(
    "org.apache.spark" %% "spark-core" % "2.0.1",
    "org.apache.spark" %% "spark-sql" % "2.0.1"
)
// Could add other dependencies here e.g.
// libraryDependencies += "org.apache.spark" %% "spark-mllib" % "2.0.1"
```

The `project.sbt` defines how our project will be compiled.
The code that is to be compiled is hidden a few directories deeper.

View it using the `more` command

```
more src/main/scala/hello.scala`
```

Which should give the output

```
import org.apache.spark.sql.SparkSession
// define main method (scala entry point)
object HelloWorld {
  def main(args: Array[String]): Unit = {

    // initialise spark session (running in "local" mode)
    val sparkSession = SparkSession.builder
      .master("local")
      .appName("Hello World")
       .getOrCreate()

    // do stuff
    println("Hello, world!")

    // terminate underlying spark context
    sparkSession.stop()
  }
}
```

### Exercise 5: Compile and run HelloWorld

We've downloaded a project, taken a look at it and all seems well. We are almost ready to compile.

The command we need to use is `sbt package` but when we try it, it doesn't work:

```
sbt package
```

results in

```
bash: sbt: command not found
```

This error message occurs because the `sbt` command is not available to us by default when we start a `qrshx` session on a compute node.

To make `sbt` available (and Java and Spark which we also need), We first have to load the relevant `module files`

```
module load apps/java/jdk1.8.0_102/binary
module load dev/sbt/0.13.13
module load apps/spark/2.1.0/gcc-4.8.5
```

Now, when you type `sbt package`, it will compile your program.

If this is successful, you'll have a file in the location `target/scala-2.11/hello_2.11-1.0.jar`.

Run with

```
spark-submit --master local[1] target/scala-2.11/hello_2.11-1.0.jar
```

### Exercise 6: Manually create the directory structure

We'll now learn how to create HelloWorld from scratch to give us practice in using Linux commands.

**Make sure you are home**

Ensure you are in your home directory by executing the command `cd` on its own. Check that you are where you expect to be using the `pwd` (print working directory) command.

The result should be

`/home/abc123`

where `abc123` will be replaced by your username.

** Create the directory structure

Start by creating the project directory. We'll call this `hello` in this case.

To create our directory, we could use the graphical user interface of MobaXterm as shown in the screen shot below

<img src="images/mobaXterm_newdir.png" />

It's much easier, however, to use the `mkdir` command

`mkdir hello`

we could then proceed to create the other directories we need one command at a time:

```
mkdir hello/src
mkdir hello/src/main
mkdir hello/src/main/scala
```

Alternatively, we could take a shortcut and the `-p switch` of `mkdir` to create the whole nested structure at once.

```
mkidr -p hello/src/main/scala
```

Linux geeks are terminally lazy so if it feels like there should be a shortcut, there probably is one

However you do it, you need to create the above 4 embedded directories.

**Create the .sbt and .scala files**

Here, we create <tt>.sbt</tt> file and <tt>.scala</tt> file on the Windows machine(By downloading them or by copying and pasting them using an editor) and then transfer them to Sharc.

Recall that the <tt>.sbt</tt> file contains the dependencies required by the program. Take a look at the <tt>.sbt</tt> file included [here](files/project.sbt) for the <tt>helloWorld</tt> program. The <tt>.scala</tt> program is also [available](files/hello.scala).

**Copy the <tt>.sbt</tt> file over to Sharc**

The <tt>.sbt</tt> file needs to be placed at the top level of the project.
You can just drag and drop it from Windows to Sharc using MobaXterm.

<img src="images/mobaxterm_transfer.png" />

**Copy the <tt>.scala</tt> file over to the HPC**

The <tt>.scala</tt> file needs to be placed in the <tt>scala</tt> directory.

**Compile and run the project**

Exactly as before, we compile and run with  `sbt package.

If this is successful, you'll have a file in the location `target/scala-2.11/hello_2.11-1.0.jar`.

Run with

```
spark-submit --master local[1] target/scala-2.11/hello_2.11-1.0.jar
```

### Exercise 7: Run a program in batch mode
