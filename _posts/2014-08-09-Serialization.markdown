---
layout: post
title: Serialization - What to choose?.
category: Coding
tags: java, serialization
year: 2014
month: 08
day: 10
published: true
summary: The key to how well a system works has a very tight link to how well the objects in the system are serialized. What all design considerations can i do.

image: Serialization.png
---

Serialization of objects impacts the performance of a service considerably. The response time for receiving/sending objects is directly linked to the time taken for serialization and deserialization. If the serialized bytes are more that impacts the time spent on the wire. The key to choose the right technology should be based on 3 factors no loss of data, serialization byte size and time taken for serialization and deserialization.

![Java Serialization](/img/posts/Serialization.png)

Serialization is the process of converting the object in memory into a byte or char stream so that it can be saved to disk or sent over the wire.

Deserialization is the process of converting a byte stream or char stream into objects.

There are two design decision we need to take while choosing the right format.

* Byte Stream

We choose byte stream to reduce the size of the object or because that is the default supported by the programming language out of the box. When we want to consider performance byte stream would be the first choice. Java Serialization will defenitely not be the right serialization platform to use as it is very slow and very descriptive. 

There are many choices to make in order to pick the right serialization API
other than java serialization.

**Google Protobuf**

    Google's protobuf is extremely fast. The way it works is by generating a custom serialization class for the object. It writes a number for each type that is serialized and the value. Hence considerbly reducing the type information that is written. Hence the binary stream written by protobuf is propereitary you will have to use the same objects builder to de-serialize the object back. Protobuf is very ideal as it supports for C++, java and other languages. Major disadvantage is, it cannot support Generic object types in a class. This makes it not suitable or would put more work on the client and server API to convert the generic objects into strong types. In which case serialization and deserialization time becomes double while designing Banyan.

**GSON and others**
    
    There are other fast binary serializtion API's like GSON in the market. These seem to mitigate the problem Protobuf has by allowing the developer to register the class before serializing it. Still we will have to write a general object serializer for the class of type Object. This makes it difficult for serializing deep objects. Out externalization code becomes very difficult.

**Fast Serialization**

    The idea behind fast serialization is impressive. Some of the impressive features of fast serialization are
    - It is a drop in replacement for java serialization and is only slightly slower than protobuf (meassured in terms of Us ). 
    - It has class preregistered for all basic objects, types and array combination. 
    - Does convert boxed objects like Long into primitive type of long while serializing. 
    - Only stores the actual allocated size for a primitive type. For ex:- if you define a type as long and its actual value is 123 it only uses 2 bytes allocation to serialize it , 1 byte for storing the type and 1 for the actual value of 123.
    - Repetitive strings serialized as part of the same object is stored only once.
    
    It almost felt like some one read my mind today and went back in time to write it ;). The data being compressed using this is smaller in size than protobuf and almost the same size as lz4 compression with more speed.
    
    It is not fair if i don't mention the disadvantages even though i like it a lot. The developer uses UnSafe methods to acheive the speed of serialization. I am trying to push it extreme to know its limitations. It is working in a fantastic way when trying to serialize and de-serialize objects of size less than 2 MB which is 10 times bigger in size when using java serialization. Performance degradation is seen when the object size goes higher, it is still able to serialize and deserialize within times lesser than java serialization.
    

Let is look at some code to see how easy it is to use FSTSerialization

***Serialization Code***

<script src="https://gist.github.com/vallur/f0f67b213f9a56a715a3.js"></script>

***De-Serialization Code***    
<script src="https://gist.github.com/vallur/0a2f938593bf5fa56812.js"></script>

In the above example the same set of bytes are used for serialization as well as de-serialization so constant known stress for GC.

Some metrics while doing serialization and deserialization between client and server on the same machine using FST and Banyan random rows fetch

![FSTSerialize](/img/posts/JMXSerialize.png)

The max processing time is because of my MAC crying and saying not enough memory.

* Char stream

The char stream way of serialization is slow as they are either verbose or evrything is converted into strings which makes them slow.

**XML**
XML is usually too verbose as it has to have start and end tags along with the name and value attributes to make it readable. Also to store in xml we will have to escape the escase chars makes the string manipulation on large blobs very slow, considering that strings are immutable in java. It is simple to read follows a strong schema and it also incurs the cost of validation of input as the different types could have garbage values and type mismaches. Easy for other languages to support.

**JSON**
Much simpler than XML and very easy to read. Too less of a charectars to escape. Easy for other languages to support. Can be holded in a huge hashmap lke object type. Since this also needs the Key and value pair combination to mostly make it readable. This also is verbose but less than XML. Time taken for serialization and deserialization is less but considerable higher than binary format.

**Table**
When trying to read or write tabular data it is a fair choice to choose tabular data where the first row is the columns and consecutive rows are values. It is way simpler than the above two to interpret. The disadvantage is not all the data we read and write is tabular in format.

***Conclusion***
I am thinking of using a combination of the three to support a wide range of audience. To start with to give a rock solid product, I can use Fast Serialization with binary stream and later while developing admin tools can have support for tabular for single row and multi row select from singe table and JSON while trying to select graph data. This would make php and javascript client's happy.

***Disclaimer***
The suggestions provided are suitable for Banyan and should not be considered as a defacto for all applications.


<div class="row">	
	<div class="span9 column">
			<p class="pull-right">{% if page.previous.url %} <a href="{{page.previous.url}}" title="Previous Post: {{page.previous.title}}"><i class="icon-chevron-left"></i></a> 	{% endif %}   {% if page.next.url %} 	<a href="{{page.next.url}}" title="Next Post: {{page.next.title}}"><i class="icon-chevron-right"></i></a> 	{% endif %} </p>  
	</div>
</div>

<div class="row">	
    <div class="span9 columns">    
		<h2>Comments Section</h2>
	    <p>Feel free to comment on the post but keep it clean and on topic.</p>	
		<div id="disqus_thread"></div>
		<script type="text/javascript">
			/* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
			var disqus_shortname = 'vallur'; // required: replace example with your forum shortname
			var disqus_identifier = '{{ page.url }}';
			var disqus_url = 'http://erjjones.github.com{{ page.url }}';
			
			/* * * DON'T EDIT BELOW THIS LINE * * */
			(function() {
				var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
				dsq.src = 'http://' + disqus_shortname + '.disqus.com/embed.js';
				(document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
			})();
		</script>
		<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
		<a href="http://disqus.com" class="dsq-brlink">blog comments powered by <span class="logo-disqus">Disqus</span></a>
	</div>
</div>

<!-- Twitter -->
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="//platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>

<!-- Google + -->
<script type="text/javascript">
  (function() {
    var po = document.createElement('script'); po.type = 'text/javascript'; po.async = true;
    po.src = 'https://apis.google.com/js/plusone.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(po, s);
  })();
</script>