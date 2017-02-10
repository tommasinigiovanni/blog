---
layout: post
title: "How to put the monit version in a bash variable"
date:   2017-02-10
categories: [linux, IT, system administrator, regex]
---

![Screenshot monit](https://mmonit.com/monit/img/logo.png "Monit Logo")
![Screenshot regex]({{ site.url }}/static/img/_posts/regex.png "Regex Image")<br/>

I was looking for a way to save in a variable, the monit version installed in my server<br/>

If I use the command `monit -V` the result is very verbose, like this:
{% highlight bash %}
This is Monit version 5.6
Copyright (C) 2001-2013 Tildeslash Ltd. All Rights Reserved.
{% endhighlight %}

Instead I would like to only `5.6`

So I write a small bash script with a [Regular Expression](http://www.regular-expressions.info/) to find and asve in a variable only that `5.6`.

{% highlight bash %}
#!/bin/bash

m_v="$(monit -V)"

for f in $m_v ; do
    if [[ $f =~ ^[+-]?[0-9]+\.?[0-9]*+\.?[0-9]*$ ]]; then
        monit_version=$f
    fi
done

echo $monit_version
{% endhighlight %}

---

## Linkography
1. [https://mmonit.com/monit/](https://mmonit.com/monit/)
2. [http://www.regular-expressions.info](http://www.regular-expressions.info)
