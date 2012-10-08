---
title: The Love-Hate of JiBX
layout: post 
summary: where I muse about JiBX, an XML-Object mapper, and whether writing binding files is any better than writing translation layers
categories: old
veryold: true
tag: XML
---

Ok, I love JiBX's flexibility.  I have two different versions of an existing schema that I'm mapping to the same Java object, which is the same Java object I'm then storing in Hibernate.  This is the first time I've been able to use the same domain object throughout the application, and I love it.  No more relying on framework-generated objects, and no more translation layers.

Framework-generated objects are bad for my schemas, which have a lot of nested anonymous complex types (hey, I didn't write them) which frameworks tend to handle badly.  XMLBeans, for instance, creates nested inner types, which gets very ugly.  It may be personal preference, but that seems very messy; it means one very large object. Hibernate, I imagine, would also be unhappy with that setup. 

So the alternative is a translation layer -- a class that takes in a WebserviceFriendlyDomainObject and spits back a HibernateFriendlyDomainObject. This is a pain to write, but mostly just smells bad -- it just shouldn't be necessary.  I've done it as a workaround so far, but it seems like there should be a way around it.

JiBX lets me use the same object from front to back. But that brings me to one of the hates -- I have to write a JiBX binding file instead.  It's funny, because for whatever reason this passes under the "ugly architecture" radar, but it's actually not much different than writing a translation layer. It's just writing your translation layer in XML instead of in Java. Which, if anything, simply adds difficulty, because I find myself trying to do complex logic in the binding file that could really benefit from a Turing-complete language. Which is ironic, since I'm writing it to save me from writing the same thing in Java. 

Now, it's still a bit better than a translation layer. The typical translator is converting one object to another object to be turned into XML. The JiBX binding is going straight to XML. It's certainly faster than the translation step. But from the coder's point of view, the XMLBeans-generated domain object is one method call away from being XML, so it doesn't <i>feel</i> substantially different.

My other complaint about JiBX so far is web service support -- it's pretty new, and support in Axis2 (another topic entirely) and XFire is either brand-new or still in CVS.  I'm finding that my data binding layer is informing my choice in web service stacks, which I resent.  I haven't tried XFire's JiBX support yet (since Codehaus's SVN server is down) but they were planning on getting it into 1.1 and it didn't make it, which isn't a good sign. 

The annoyances of bytecode manipulation are already documented elsewhere; I've found that every now and again I have to run an ant task that inserts the JiBX bindings into Eclipse's class files, and that's the extent of it. It irks some of people, but doesn't bother me. 

So I'm not sure if JiBX will stay around in this project. That's in spite of substantial positives -- at this point, it could easily fall either way. 

<i>Update, 2006-05-16:</i>

It didn't stay around. In the end, expressing logic ("if this element... else if this element...") in XML was too much. I moved over to a hand-rolled STaX serializer and deserializer -- see ["for the record"](/2006/05/16/for-the-record.html) for a quick peek into the main downside there. 

Why I went this route instead of XMLBeans or JAXB merits more discussion.  Briefly, JAXB didn't meet the performance requirements for this part of the system, and XMLBeans refuses to fully parse the schemas in question. It tries, but something in the deeply nested anonymous complex types gives it fits.  Writing code against those types as they're generated in XMLBeans is also ugly, but then so is the custom parser. 

All in all, I'm coming to terms with the translation layer. But it still feels like it oughtn't be necessary...  
