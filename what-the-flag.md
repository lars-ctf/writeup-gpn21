# What-the-flag

The provided webpage tells us that the "KITCTF FlagGenerator" is
currently not available to the public. There are no buttons or links,
leading us to investigate the source-code.

We found the comment
`<!-- TODO prevent google from leaking our source code -->` in the
source code. Well, how can you prevent Google (or web crawlers in
general) from crawling your webpage? Using the `/robots.txt`!

The `robots.txt` file contains the string
`User-agent: * Disallow: /source`. Soo the next step was obviously to
investigate the `/source` file. This was the moment we understood the
name of the challenge "wtf" -- the code mainly consists of "(", ")", "="
and "+", all wrapped in an `exec` call. The existence of the `exec`
function made us believe that the source is written in Python (lucky
guess, to be honest). As the string inside the `exec` function would
represent some kind of source-code, we could potentially retrieve it by
replacing it with a `print` function -- which gave us a (weirdly
formatted) Flask backend!

Two routes were especially interesting:

``` python
@app.route('/3ng1n33r1ng-s4mpl3',methods=['POST'])
def flag():
    A={A[0]:A[1]for A in request.headers.items()};B=request.args.to_dict()
    if subprocess.check_output(['node',_A,str(A),str(B),str(request.json)]).decode().strip()=='Valid':return os.environ.get('FLAG')
    return redirect('/')
```

and

``` python
@app.route('/source/js')
def js_source():
    with open(_A)as A:return A.read()
```

After some investigation it seems like POSTing some data to
`/3ng1n33r1ng-s4mpl3` would return the flag *if* the provided JS code
(applied with the given request data) would return "valid".

The `/source/js` code, however, looks even weirder than the previous
Python code.

``` javascript
[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]](([][[]]+[])[!+[]+!+[]]+([]+[])[(![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])...
```

Luckily we immediatly realized that this encoding must be "jsfuck". By
executing and reformatting the code, we arrived at a NodeJS application
that uses its arguments to decide whether they match the interpretation
of some brainfuck codes (wtf?):

``` javascript
const headers = JSON.parse(process.argv[2].split("'").join('"')),
  args = JSON.parse(process.argv[3].split("'").join('"')),
  body = JSON.parse(process.argv[4].split("'").join('"'));
headers[interpret(d.header_key)] === interpret(d.header_value) &&
args[interpret(d.query_key)] === interpret(d.query_value) &&
body[interpret(d.body_key)] === interpret(d.body_value) &&
headers.Cookie.split("=")[0] === interpret(d.cookie_key) &&
headers.Cookie.split("=")[1] === interpret(d.cookie_value)
  ? console.log("Valid")
  : console.log("Fake");
```

By calling the `interpret` function directly on the values in the `d`
object and logging the results, we achieved the data that we would have
to put in the request parameters:

-   path "/3ng1n33r1ng-s4mpl3"
-   cookie with "nda=true"
-   json body with {"resource": "flag"}
-   header with "X-Early-Access: kitctf-certified-tester"
-   query with "key=5468697320697320612076616c6964206b6579"

Plugging all this into a `curl` request finally gave us the flag:

``` bash
curl --cookie "nda=true" --data '{"resource":"flag"}' -H "X-Early-Access: kitctf-certified-tester" -H "Content-Type: application/json" -X POST "https://wtf-0.chals.kitctf.de/3ng1n33r1ng-s4mpl3?key=5468697320697320612076616c6964206b6579"
```
