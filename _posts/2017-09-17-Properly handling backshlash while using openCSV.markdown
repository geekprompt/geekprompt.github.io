---
layout: post
title:  "Properly handling backslashes using OpenCSV"
date:   2017-09-17 14:36:00 +0530
categories: csv, java, trouble-shooting
comments: true
---

OpenCSV is one of the popular JAVA libraries used for handling CSV data. In this post
I will discuss about one specific issue which we recently faced with this library.

**The Problem:**

Here is a minimal code snippet for writing and reading CSV data using OpenCSV.


{% highlight java %}

String dataValue = "test";

//writing  
StringWriter writer = new StringWriter();

try (CSVWriter csvwriter = new CSVWriter(writer)) {
    String[] originalData = new String[2];
    originalData[0] = dataValue;
    originalData[1] = dataValue;
    System.out.println("Original data: " + originalData[0] + "," + originalData[1]);
    csvwriter.writeNext(originalData);
} catch (IOException e) {
    throw new RuntimeException(e);
}
System.out.println("Written data: " + writer.toString());

//reading
try (CSVReader csvReader = new CSVReader(new StringReader(writer.toString()))) {
          String[] readData = csvReader.readNext();
          System.out.println("Read data: " + readData[0] + "," + readData[1]);
      } catch (IOException e) {
          throw new RuntimeException(e);
      }
{% endhighlight %}

The output of the above snippet:

{% highlight text %}
Original data: test,test
Written data: "test","test"

Read data: test,test
{% endhighlight %}


Which is as expected. Well, the life is good with OpenCSV until you encounter
a backslash character ('\\') in your CSV data.

So let's try running the same snippet with `dataValue` having a backslash character:

{% highlight java %}
String dataValue = "t\\est";
{% endhighlight %}

Output:

{% highlight text %}
Original data: t\est,t\est
Written data: "t\est","t\est"

Read data: test,test
{% endhighlight %}

Note that the backslash character is gone in the read CSV data.

**The root cause:**

By default `CSVReader` is using backslash ('\\') as escape character. Whereas
`CSVWriter` is using a double quote('"') as escape character.

Because of this at the time of writing the data backslash characters are not
properly escaped. At the time of reading, a single backslash character will be
ignored by the `CSVParser` as it is the escape character.

**The Solution:**

By default `CSVReader` uses `CSVParser` which for parsing CSV data. OpenCSV
provides another parser (`RFC4180Parser`) which strictly follows RFC4180 standards.

Using with `RFC4180Parser`, the `CSVReader` will use double quote('"') as the
escape character making it consistent with `CSVWriter`.

We need to replace the _reading_ part of above mentioned snippet with following code:

{% highlight java %}
RFC4180Parser rfc4180Parser = new RFC4180ParserBuilder().build();
CSVReaderBuilder csvReaderBuilder = new CSVReaderBuilder(new StringReader(writer.toString()))
                .withCSVParser(rfc4180Parser);
try (CSVReader csvReader = csvReaderBuilder.build()) {
    String[] readData = csvReader.readNext();
    System.out.println("Read data: " + readData[0] + "," + readData[1]);
} catch (IOException e) {
    throw new RuntimeException(e);
}
{% endhighlight %}

Output:
{% highlight text %}
Original data: t\est,t\est
Written data: "t\est","t\est"

Read data: t\est,t\est
{% endhighlight %}

If you are looking to change the library itself then [Apache commons CSV](https://commons.apache.org/proper/commons-csv/) is a good alternative for
OpenCSV.

If you have any suggestions and/or queries related to this post then please
start a discussion in the comment section below.

**Library Version used for the code snippets:**

[OpenCSV 4.0](https://mvnrepository.com/artifact/com.opencsv/opencsv/4.0){:target="_blank"}

_**References:**_

[sourceforge support request](https://sourceforge.net/p/opencsv/support-requests/50/){:target="_blank"}

[RFC4180](https://tools.ietf.org/html/rfc4180){:target="_blank"}

[OpenCSV official page](http://opencsv.sourceforge.net/){:target="_blank"}
