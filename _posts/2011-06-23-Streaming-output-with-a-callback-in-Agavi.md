---
layout: post
title: Streaming output with a callback in Agavi
---

{{ page.title }}
================

<a href="/2011/01/15/Serving-large-xml-files.html">Earlier</a> I wrote about serving large XML contents with PHP. Today I wanted to  use that knowledge when outputting from Agavi.
By default you would output XML like:

{% highlight php %}
class Products_ListSuccessView extends MyProductsBaseView
{
    
    public function executeXml(AgaviRequestDataHolder $rd)
    {
        $dom = new DOMDocument();
        ........
        return $dom->saveXML();
    }
}
{% endhighlight %}

Changing this according to my previous blog post about XML isn’t just replacing the DOMDocument with XmlWriter as AgaviWebResponse sends headers only after the view’s execute-method has completed.

So I had to subclass AgaviWebResponse to support returning Closures from the view:

{% highlight php %}
class MyWebResponse extends AgaviWebResponse
{
    public function sendContent()
    {
        if ($this->content instanceof Closure) {
            $func = $this->content;
            $func();
            return;
        }
        parent::sendContent();
    }
}
{% endhighlight %}

After adding the class to autoload.xml and factories.xml you can do the following:

{% highlight %}
class Products_ListSuccessView extends MyProductsBaseView
{
    
    public function executeXml(AgaviRequestDataHolder $rd)
    {
        return function() {
            $writer = new XmlWriter();
            $writer->openURI('php://output');
            .......
            $writer->flush();
        };
    }
}
{% endhighlight %}

Wasn’t that simple? :)