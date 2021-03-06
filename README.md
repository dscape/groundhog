# groundhog

A tool to allow recording and playback of API responses to allow for predictable automated testing

## Installation

```
npm install -g groundhog
```

## Usage

```
Usage: groundhog
Attempts to load configuration options from a ".groundhog.json" if it exists in the current working directory.


Usage: groundhog --config
--config        load the options from a config file specified


Usage: groundhog --hostname <hostname> --dir <directory> [--port] [--record] [--strict]
--hostname      host to which to proxy requests
--dir           directory in which to read/write playback files
--record        record the current session
--port          port to run on (default: 3001)
--protocol      protocol to run (default: http)
--strict        only serve recordings in playback mode
```

## Sample

``` sh
cd sample
npm install
```

You are running a simple transparent proxy from your server to a backend
that is defined in config.js or by environment variable.

``` sh
# what am i running?
cat app.js
```

Our `.groundhog.json` is where we define where we want to store the tests,
and proxies we want to use.

In this case we are saving the recordings to `./test/recordings` and proxying
anything that goes against port `3003` to the `yld.io` website

``` sh
cat .groundhog.json
```

``` js
{
  "dir": "./test/recordings",
  "proxies": [
     {  "hostname": "yld.io",
        "port": 3003,
        "protocol": "http"
     }
  ]
}
```

We can now use environment variables to change our website to connect to our
proxy, diverting the traffic that normally would go directly against the backend
 (see `test/run`):

``` sh
_PROXYTO=http://localhost:3003 _APPPORT=1337 node app.js
```

Our test is simple, it runs a request and checks for errors. It uses the same
config file so as long as your environments variables are set right this will
consistently

``` sh
# what am i testing?
cat test/index.js
```

If you run the test with our script you will get a lot of warnings, we know
you are trying to run mocked tests and we couldn't find any mocks. Setting
`"strict": true` in your `.groundhog.json` file will make this throw.

This is useful since these interactions are sometimes meaningful, and if
a previous request was not executed (eg. login) the subsequent requests might
not work properly.

``` sh
# run tests
./test/run
```

If you record the results, and run again, it should be out of warnings.
Everything was returned from mocks.

``` sh
# record test results
./test/run --record
ls test/recordings
# run tests using recordings
./test/run
```

Of course, don't forget to run the real tests. If they break and mocks don't,
you might have a problem.
