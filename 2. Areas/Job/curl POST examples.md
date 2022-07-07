

<h2>Common Options</h2>
<div>-#, --progress-bar Make curl display a simple progress bar instead of the more informational standard meter.</div>
<div>-b, --cookie &lt;name=data&gt; Supply cookie with request. If no =, then specifies the cookie file to use (see -c).</div>
<div>-c, --cookie-jar &lt;file name&gt; File to save response cookies to.</div>
<div>-d, --data &lt;data&gt; Send specified data in POST request. Details provided below.</div>
<div>-f, --fail Fail silently (don't output HTML error form if returned).</div>
<div>-F, --form &lt;name=content&gt; Submit form data.</div>
<div>-H, --header &lt;header&gt; Headers to supply with request.</div>
<div>-i, --include Include HTTP headers in the output.</div>
<div>-I, --head Fetch headers only.</div>
<div>-k, --insecure Allow insecure connections to succeed.</div>
<div>-L, --location Follow redirects.</div>
<div>-o, --output &lt;file&gt; Write output to . Can use --create-dirs in conjunction with this to create any directories specified in the -o path.</div>
<div>-O, --remote-name Write output to file named like the remote file (only writes to current directory).</div>
<div>-s, --silent Silent (quiet) mode. Use with -S to force it to show errors.</div>
<div>-v, --verbose Provide more information (useful for debugging).</div>
<div>-w, --write-out &lt;format&gt; Make curl display information on stdout after a completed transfer. See man page for more details on available variables. Convenient way to force curl to append a newline to output: -w "\n" (can add to ~/.curlrc).</div>
<div>-X, --request The request method to use.</div>
<h2>POST</h2>
<div>When sending data via a POST or PUT request, two common formats (specified via the Content-Type header) are:</div>
<ul><li><div>application/json</div></li><li><div>application/x-www-form-urlencoded</div></li></ul>
<div>Many APIs will accept both formats, so if you're using curl at the command line, it can be a bit easier to use the form urlencoded format instead of json because</div>
<ul><li><div>the json format requires a bunch of extra quoting</div></li><li><div>curl will send form urlencoded by default, so for json the Content-Type header must be explicitly set</div></li></ul>
<div>This gist provides examples for using both formats, including how to use sample data files in either format with your curl requests.</div>
<h2>curl usage</h2>
<div>For sending data with POST and PUT requests, these are common curl options:</div>
<ul><li><div>request type</div></li><ul><li><div>-X POST</div></li><li><div>-X PUT</div></li></ul><li><div>content type header</div></li><li><div>-H "Content-Type: application/x-www-form-urlencoded"</div></li><li><div>-H "Content-Type: application/json"</div></li><li><div>data</div></li><ul><li><div>form urlencoded: -d "param1=value1&amp;m2=value2" or -d @data.txt</div></li><li><div>json: -d '{"key1":"value1", "key2":"value2"}' or -d @data.json</div></li></ul></ul>
<h2>Examples</h2>
<h3>POST application/x-www-form-urlencoded</h3>
<div>application/x-www-form-urlencoded is the default:</div>
<div>curl -d "param1=value1&amp;m2=value2" -X POST http://localhost:3000/data</div>
<div>explicit:</div>
<div>curl -d "param1=value1&amp;m2=value2" -H "Content-Type: application/x-www-form-urlencoded" -X POST http://localhost:3000/data</div>
<div>with a data file</div>
<div>curl -d "@data.txt" -X POST http://localhost:3000/data</div>
<h3>POST application/json</h3>
<div>curl -d '{"key1":"value1", "key2":"value2"}' -H "Content-Type: application/json" -X POST http://localhost:3000/data</div>
<div>with a data file</div>
<div>curl -d "@data.json" -X POST http://localhost:3000/data</div>
<div><br></div>