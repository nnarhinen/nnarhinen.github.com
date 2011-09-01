---
layout: post
title: Creating and Serving large XML files - DOMDocument vs. XmlWriter
---

{{ page.title }}
================

Handling big amounts of data between a client application and a REST service can sometimes be tricky, especially when things should be responsive. I did a small benchmark between the built-in tools in PHP: DOMDocument and XmlReader.

## DOMDocument

DOMDocument is maybe the most used method for XML handling and creation in PHP. The approach used in DOMDocument, is to load nodes into a DOM, which then can be queried and/or represented as a string.

Here's a sample script I used for benchmarking:

{% highlight php %}
<?php
$start = microtime(true);
header('Cache-Control: no-cache, must-revalidate');
header('Expires: Mon, 26 Jul 1997 05:00:00 GMT');
header('Content-type: text/xml');
$doc = new DOMDocument('1.0', 'utf-8');
$doc->formatOutput = true;
$products = $doc->createElement('Products');
for ($i = 0; $i < 1000000; $i++) {
    $element = $doc->createElement('Product');
    $element->setAttribute('Id', $i);
    $element->setAttribute('Value', md5(uniqid()));
    $products->appendChild($element);
}
$doc->appendChild($products);
echo $doc->saveXml();

$stop = microtime(true);
$seconds = $stop - $start;
echo "Start: " . $start . PHP_EOL;
echo "Stop: " . $stop . PHP_EOL;
echo "Seconds: " . $seconds . PHP_EOL;
echo "Memory peak: " . memory_get_peak_usage() / 1048576 . 'MB' . PHP_EOL;
?>
{% endhighlight %}

This script was served via Apache and downloaded with wget.

Here are some stats:

 * The average time to create the xml and output it, was somewhere near 30-35 seconds.
 * Memory consumption average was 120-130MB.
 * The average time to complete the whole wget request, was somewhere between 35-40 seconds.

## XmlWriter

XmlWriter is not that well known amongst PHP-developers. It's basically a wrapper for libxml's xmlWriter API. With XmlWriter you can (in theory, at least) flush the contents to the output-pipe before you have even processed all data on your server. That way you can start processing the data clientside before the download is even complete. The processing and 'editing' capabilities are a bit more limited than with DOMDocument, and you need to handle the document hierarchy yourself.

Here's a sample I wrote:

{% highlight php %}
<?php
$start = microtime(true);
header('Cache-Control: no-cache, must-revalidate');
header('Expires: Mon, 26 Jul 1997 05:00:00 GMT');
header('Content-type: text/xml');

$writer = new XmlWriter();
$writer->openURI('php://output');
$writer->startDocument('1.0', 'utf-8');
$writer->startElement('Products');
$writer->setIndent(true);
for ($i = 0; $i < 1000000; $i++) {
    $writer->startElement('Product');
    $writer->writeAttribute('Id', $i);
    $writer->writeAttribute('Value', md5(uniqid()));
    $writer->endElement();
}
$writer->endElement();
$writer->flush();

$stop = microtime(true);
$seconds = $stop - $start;
echo "Start: " . $start . PHP_EOL;
echo "Stop: " . $stop . PHP_EOL;
echo "Seconds: " . $seconds . PHP_EOL;
echo "Memory peak: " . memory_get_peak_usage() / 1048576 . 'MB' . PHP_EOL;
?>
{% endhighlight %}

This script was as well served via Apache and requested with wget.

Here are some stats on XmlWriter performance:

 * The average time to create the xml and output it, was somewhere near 16-20 seconds
 * Memory consumption average was somewhere between 500-700 kb
 * The whole wget request took only the same 16-20 seconds

## And the winner is: XmlWriter

So to sum things up. If the capabilities XmlWriter offers are enough for you, you should definitely be using it. With one million 'product' items, the times to process the data and get the contents to your client will be split in half comparing to DOMDocument. And what's most important: you will be consuming only a fraction of the memory, which then leads to better performance and scalability.