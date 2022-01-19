---
layout: post
title: "Janino compiler for on-the-fly dynamic code compilation"
date: 2022-01-18 12:57:23 +0530
tags: [java]
comments: true
---
I got to know about Janino when I got the opportunity to work on [Apache Spark](https://spark.apache.org/){:target="_blank"} internals at one of my previous organizations.

With the release of version 2.0, Apache Spark started using _Whole-stage Code Generation_ instead of _Volcano Iterator Model_. Covering the details of
_Whole-stage Code Generation_ and _Volcano Iterator Model_ will be out of scope for this post, however,
 [this blog post](https://databricks.com/blog/2016/05/23/apache-spark-as-a-compiler-joining-a-billion-rows-per-second-on-a-laptop.html){:target="_blank"} explains both these concepts in detail.

The _Whole-stage Code Generator_ generates Java code for a specific stage of a Spark job and Spark uses Janino to compile this dynamically generated
 code on-the-fly.

The latest version of the Janino compiler maven package can be found [here](https://mvnrepository.com/artifact/org.codehaus.janino/janino) and can be
 included as part of your project.

The following code demonstrates the compilation of a _Hello World_ Java program using Janino:

{% highlight java %}

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import org.codehaus.commons.compiler.CompileException;
import org.codehaus.janino.ClassBodyEvaluator;

public class HelloWorldJanino {

    public static void main(String[] args) throws CompileException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {

        String codeString = "public static void main(String[] args) {" +
                            "   System.out.println(\"Hello World!\" );" +
                            "}";

        // Compile the class body, and get the loaded class
        Class<?> cl = new ClassBodyEvaluator(codeString).getClazz();

        // Invoke the "public static main(String[])" method
        Method mainMeth = cl.getMethod("main", String[].class);

        mainMeth.invoke(null, new Object[]{new String[]{}});
    }
}

{% endhighlight %}

Link to source code: [https://github.com/geekprompt/code-examples/tree/master/janino](https://github.com/geekprompt/code-examples/tree/master/janino){:target="_blank"}

_**References:**_

[Janino compiler official site](https://janino-compiler.github.io/janino/){:target="_blank"}  
[Apache Spark as a Compiler: Joining a Billion Rows per Second on a Laptop](https://databricks.com/blog/2016/05/23/apache-spark-as-a-compiler-joining-a-billion-rows-per-second-on-a-laptop.html){:target="_blank"}
