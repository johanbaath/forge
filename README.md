This branch is only slightly different from b2da661d, yet this one works with PhantomJS and the other one does not.

See [comparison in Github](https://github.com/digitalbazaar/forge/compare/b2da661d...ysangkok:phantom_still_works?expand=1)

















# Forge

[![npm package](https://nodei.co/npm/node-forge.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/node-forge/)

[![Build status](https://img.shields.io/travis/digitalbazaar/forge.svg?branch=master)](https://travis-ci.org/digitalbazaar/forge)

A native implementation of [TLS][] (and various other cryptographic tools) in
[JavaScript][].

## Introduction

The Forge software is a fully native implementation of the [TLS][] protocol in
JavaScript as well as a set of tools for developing Web Apps that utilize many
network resources.

## Performance

Forge is fast. Benchmarks against other popular JavaScript cryptography
libraries can be found here:

http://dominictarr.github.io/crypto-bench/

http://cryptojs.altervista.org/test/simulate-threading-speed_test.html

## Getting Started
------------------

### Node.js ###

If you want to use forge with [node.js][], it is available through `npm`:

https://npmjs.org/package/node-forge

Installation:

    npm install node-forge

You can then use forge as a regular module:

    var forge = require('node-forge');

### Requirements ###

* General
  * Optional: GNU autotools for the build infrastructure if using Flash.
* Building a Browser Bundle:
  * nodejs
  * npm
* Testing
  * nodejs
  * Optional: Python and OpenSSL development environment to build
  * a special SSL module with session cache support for testing with flash.
  * http://www.python.org/dev/
  * http://www.openssl.org/
  * Debian users should install python-dev and libssl-dev.
* Optional: Flash
  * A pre-built SocketPool.swf is included.
  * Adobe Flex 3 SDK to build the Flash socket code.
  * http://opensource.adobe.com/wiki/display/flexsdk/

### Building a browser bundle ###

To create a minimized JavaScript bundle, run the following:

```
npm install
npm run minify
```

This will create a single minimized file that can be included in
the browser:

```
js/forge.min.js
```

Include the file via:

```html
<script src="js/forge.min.js"></script>
```

Note that the minify script depends on the requirejs package,
and that the requirejs binary 'r.js' assumes that the name of
the node binary is 'node' not 'nodejs', as it is on some
systems. You may need to change the hashbang line to use
'nodejs' or run the command manually.

To create a single non-minimized file that can be included in
the browser:

```
npm install
npm run bundle
```

This will create:

```
js/forge.bundle.js
```

Include the file via:

```html
<script src="js/forge.bundle.js"></script>
```

The above bundles will synchronously create a global 'forge' object.

Keep in mind that these bundles will not include any WebWorker
scripts (eg: prime.worker.js) or their dependencies, so these will
need to be accessible from the browser if any WebWorkers are used.

### Testing with NodeJS & RequireJS ###

A test server for [node.js][] can be found at `./nodejs`. The following are included:

  * Example of how to use `forge` within NodeJS in the form of a [mocha](http://mochajs.org/) test.
  * Example of how to serve `forge` to the browser using [RequireJS](http://requirejs.org/).

To run:

    cd nodejs
    npm install
    npm test
    npm start


### Old build system that includes flash support ###

To build the whole project, including Flash, run the following:

    $ ./build-setup
    $ make

This will create the SWF, symlink all the JavaScript files, and build a Python
SSL module for testing. To see configure options, run `./configure --help`.

### Old test system including flash support ###

A test server is provided which can be run in TLS mode and non-TLS mode. Use
the --help option to get help for configuring ports. The server will print out
the local URL you can vist to run tests.

Some of the simplier tests should be run with just the non-TLS server::

    $ ./tests/server.py

More advanced tests need TLS enabled::

    $ ./tests/server.py --tls

## Contributing
---------------

Any contributions (eg: PRs) that are accepted will be brought under the same
license used by the rest of the Forge project. This license allows Forge to
be used under the terms of either the BSD License or the GNU General Public
License (GPL) Version 2.

See: [LICENSE](https://github.com/digitalbazaar/forge/blob/cbebca3780658703d925b61b2caffb1d263a6c1d/LICENSE)

If a contribution contains 3rd party source code with its own license, it
may retain it, so long as that license is compatible with the Forge license.

## Documentation
----------------

### Transports

* [TLS](#tls)
* [HTTP](#http)
* [SSH](#ssh)
* [XHR](#xhr)
* [Sockets](#socket)

### Ciphers

* [CIPHER](#cipher)
* [AES](#aes)
* [DES](#des)
* [RC2](#rc2)

### PKI

* [RSA](#rsa)
* [RSA-KEM](#rsakem)
* [X.509](#x509)
* [PKCS#5](#pkcs5)
* [PKCS#7](#pkcs7)
* [PKCS#8](#pkcs8)
* [PKCS#10](#pkcs10)
* [PKCS#12](#pkcs12)
* [ASN.1](#asn)

### Message Digests

* [SHA1](#sha1)
* [SHA256](#sha256)
* [SHA384](#sha384)
* [SHA512](#sha512)
* [MD5](#md5)
* [HMAC](#hmac)

### Utilities

* [Prime](#prime)
* [PRNG](#prng)
* [Tasks](#task)
* [Utilities](#util)
* [Logging](#log)
* [Debugging](#debug)
* [Flash Socket Policy Module](#fsp)

---------------------------------------

If at any time you wish to disable the use of native code, where available,
for particular forge features like its secure random number generator, you
may set the ```disableNativeCode``` flag on ```forge``` to ```true```. It
is not recommended that you set this flag as native code is typically more
performant and may have stronger security properties. It may be useful to
set this flag to test certain features that you plan to run in environments
that are different from your testing environment.

To disable native code when including forge in the browser:

```js
forge = {disableNativeCode: true};
// now include forge script file(s)
// Note: with this approach, script files *must*
// be included after initializing the global forge var

// alternatively, include script files first and then call
forge = forge({disableNativeCode: true});

// Note: forge will be permanently reconfigured now;
// to avoid this but use the same "forge" var name,
// you can wrap your code in a function to shadow the
// global var, eg:
(function(forge) {
  // ...
})(forge({disableNativeCode: true}));
```

To disable native code when using node.js:

```js
var forge = require('node-forge')({disableNativeCode: true});
```

---------------------------------------
## Transports

<a name="tls" />
### TLS

Provides a native javascript client and server-side [TLS][] implementation.

__Examples__

```js
// create TLS client
var client = forge.tls.createConnection({
  server: false,
  caStore: /* Array of PEM-formatted certs or a CA store object */,
  sessionCache: {},
  // supported cipher suites in order of preference
  cipherSuites: [
    forge.tls.CipherSuites.TLS_RSA_WITH_AES_128_CBC_SHA,
    forge.tls.CipherSuites.TLS_RSA_WITH_AES_256_CBC_SHA],
  virtualHost: 'example.com',
  verify: function(connection, verified, depth, certs) {
    if(depth === 0) {
      var cn = certs[0].subject.getField('CN').value;
      if(cn !== 'example.com') {
        verified = {
          alert: forge.tls.Alert.Description.bad_certificate,
          message: 'Certificate common name does not match hostname.'
        };
      }
    }
    return verified;
  },
  connected: function(connection) {
    console.log('connected');
    // send message to server
    connection.prepare(forge.util.encodeUtf8('Hi server!'));
    /* NOTE: experimental, start heartbeat retransmission timer
    myHeartbeatTimer = setInterval(function() {
      connection.prepareHeartbeatRequest(forge.util.createBuffer('1234'));
    }, 5*60*1000);*/
  },
  /* provide a client-side cert if you want
  getCertificate: function(connection, hint) {
    return myClientCertificate;
  },
  /* the private key for the client-side cert if provided */
  getPrivateKey: function(connection, cert) {
    return myClientPrivateKey;
  },
  tlsDataReady: function(connection) {
    // TLS data (encrypted) is ready to be sent to the server
    sendToServerSomehow(connection.tlsData.getBytes());
    // if you were communicating with the server below, you'd do:
    // server.process(connection.tlsData.getBytes());
  },
  dataReady: function(connection) {
    // clear data from the server is ready
    console.log('the server sent: ' +
      forge.util.decodeUtf8(connection.data.getBytes()));
    // close connection
    connection.close();
  },
  /* NOTE: experimental
  heartbeatReceived: function(connection, payload) {
    // restart retransmission timer, look at payload
    clearInterval(myHeartbeatTimer);
    myHeartbeatTimer = setInterval(function() {
      connection.prepareHeartbeatRequest(forge.util.createBuffer('1234'));
    }, 5*60*1000);
    payload.getBytes();
  },*/
  closed: function(connection) {
    console.log('disconnected');
  },
  error: function(connection, error) {
    console.log('uh oh', error);
  }
});

// start the handshake process
client.handshake();

// when encrypted TLS data is received from the server, process it
client.process(encryptedBytesFromServer);

// create TLS server
var server = forge.tls.createConnection({
  server: true,
  caStore: /* Array of PEM-formatted certs or a CA store object */,
  sessionCache: {},
  // supported cipher suites in order of preference
  cipherSuites: [
    forge.tls.CipherSuites.TLS_RSA_WITH_AES_128_CBC_SHA,
    forge.tls.CipherSuites.TLS_RSA_WITH_AES_256_CBC_SHA],
  // require a client-side certificate if you want
  verifyClient: true,
  verify: function(connection, verified, depth, certs) {
    if(depth === 0) {
      var cn = certs[0].subject.getField('CN').value;
      if(cn !== 'the-client') {
        verified = {
          alert: forge.tls.Alert.Description.bad_certificate,
          message: 'Certificate common name does not match expected client.'
        };
      }
    }
    return verified;
  },
  connected: function(connection) {
    console.log('connected');
    // send message to client
    connection.prepare(forge.util.encodeUtf8('Hi client!'));
    /* NOTE: experimental, start heartbeat retransmission timer
    myHeartbeatTimer = setInterval(function() {
      connection.prepareHeartbeatRequest(forge.util.createBuffer('1234'));
    }, 5*60*1000);*/
  },
  getCertificate: function(connection, hint) {
    return myServerCertificate;
  },
  getPrivateKey: function(connection, cert) {
    return myServerPrivateKey;
  },
  tlsDataReady: function(connection) {
    // TLS data (encrypted) is ready to be sent to the client
    sendToClientSomehow(connection.tlsData.getBytes());
    // if you were communicating with the client above you'd do:
    // client.process(connection.tlsData.getBytes());
  },
  dataReady: function(connection) {
    // clear data from the client is ready
    console.log('the client sent: ' +
      forge.util.decodeUtf8(connection.data.getBytes()));
    // close connection
    connection.close();
  },
  /* NOTE: experimental
  heartbeatReceived: function(connection, payload) {
    // restart retransmission timer, look at payload
    clearInterval(myHeartbeatTimer);
    myHeartbeatTimer = setInterval(function() {
      connection.prepareHeartbeatRequest(forge.util.createBuffer('1234'));
    }, 5*60*1000);
    payload.getBytes();
  },*/
  closed: function(connection) {
    console.log('disconnected');
  },
  error: function(connection, error) {
    console.log('uh oh', error);
  }
});

// when encrypted TLS data is received from the client, process it
server.process(encryptedBytesFromClient);
```

Connect to a TLS server using node's net.Socket:

```js
var socket = new net.Socket();

var client = forge.tls.createConnection({
  server: false,
  verify: function(connection, verified, depth, certs) {
    // skip verification for testing
    console.log('[tls] server certificate verified');
    return true;
  },
  connected: function(connection) {
    console.log('[tls] connected');
    // prepare some data to send (note that the string is interpreted as
    // 'binary' encoded, which works for HTTP which only uses ASCII, use
    // forge.util.encodeUtf8(str) otherwise
    client.prepare('GET / HTTP/1.0\r\n\r\n');
  },
  tlsDataReady: function(connection) {
    // encrypted data is ready to be sent to the server
    var data = connection.tlsData.getBytes();
    socket.write(data, 'binary'); // encoding should be 'binary'
  },
  dataReady: function(connection) {
    // clear data from the server is ready
    var data = connection.data.getBytes();
    console.log('[tls] data received from the server: ' + data);
  },
  closed: function() {
    console.log('[tls] disconnected');
  },
  error: function(connection, error) {
    console.log('[tls] error', error);
  }
});

socket.on('connect', function() {
  console.log('[socket] connected');
  client.handshake();
});
socket.on('data', function(data) {
  client.process(data.toString('binary')); // encoding should be 'binary'
});
socket.on('end', function() {
  console.log('[socket] disconnected');
});

// connect to google.com
socket.connect(443, 'google.com');

// or connect to gmail's imap server (but don't send the HTTP header above)
//socket.connect(993, 'imap.gmail.com');
```

<a name="http" />
### HTTP

Provides a native [JavaScript][] mini-implementation of an http client that
uses pooled sockets.

__Examples__

```js
// create an HTTP GET request
var request = forge.http.createRequest({method: 'GET', path: url.path});

// send the request somewhere
sendSomehow(request.toString());

// receive response
var buffer = forge.util.createBuffer();
var response = forge.http.createResponse();
var someAsyncDataHandler = function(bytes) {
  if(!response.bodyReceived) {
    buffer.putBytes(bytes);
    if(!response.headerReceived) {
      if(response.readHeader(buffer)) {
        console.log('HTTP response header: ' + response.toString());
      }
    }
    if(response.headerReceived && !response.bodyReceived) {
      if(response.readBody(buffer)) {
        console.log('HTTP response body: ' + response.body);
      }
    }
  }
};
```

<a name="ssh" />
### SSH

Provides some SSH utility functions.

__Examples__

```js
// encodes (and optionally encrypts) a private RSA key as a Putty PPK file
forge.ssh.privateKeyToPutty(privateKey, passphrase, comment);

// encodes a public RSA key as an OpenSSH file
forge.ssh.publicKeyToOpenSSH(key, comment);

// encodes a private RSA key as an OpenSSH file
forge.ssh.privateKeyToOpenSSH(privateKey, passphrase);

// gets the SSH public key fingerprint in a byte buffer
forge.ssh.getPublicKeyFingerprint(key);

// gets a hex-encoded, colon-delimited SSH public key fingerprint
forge.ssh.getPublicKeyFingerprint(key, {encoding: 'hex', delimiter: ':'});
```

<a name="xhr" />
### XHR

Provides an XmlHttpRequest implementation using forge.http as a backend.

__Examples__

```js
```

<a name="socket" />
### Sockets

Provides an interface to create and use raw sockets provided via Flash.

__Examples__

```js
```

---------------------------------------
## Ciphers

<a name="cipher" />
### CIPHER

Provides a basic API for block encryption and decryption. There is built-in
support for the ciphers: [AES][], [3DES][], and [DES][], and for the modes
of operation: [ECB][], [CBC][], [CFB][], [OFB][], [CTR][], and [GCM][].

These algorithms are currently supported:

* AES-ECB
* AES-CBC
* AES-CFB
* AES-OFB
* AES-CTR
* AES-GCM
* 3DES-ECB
* 3DES-CBC
* DES-ECB
* DES-CBC

When using an [AES][] algorithm, the key size will determine whether
AES-128, AES-192, or AES-256 is used (all are supported). When a [DES][]
algorithm is used, the key size will determine whether [3DES][] or regular
[DES][] is used. Use a [3DES][] algorithm to enforce Triple-DES.

__Examples__

```js
// generate a random key and IV
// Note: a key size of 16 bytes will use AES-128, 24 => AES-192, 32 => AES-256
var key = forge.random.getBytesSync(16);
var iv = forge.random.getBytesSync(16);

/* alternatively, generate a password-based 16-byte key
var salt = forge.random.getBytesSync(128);
var key = forge.pkcs5.pbkdf2('password', salt, numIterations, 16);
*/

// encrypt some bytes using CBC mode
// (other modes include: ECB, CFB, OFB, CTR, and GCM)
var cipher = forge.cipher.createCipher('AES-CBC', key);
cipher.start({iv: iv});
cipher.update(forge.util.createBuffer(someBytes));
cipher.finish();
var encrypted = cipher.output;
// outputs encrypted hex
console.log(encrypted.toHex());

// decrypt some bytes using CBC mode
// (other modes include: CFB, OFB, CTR, and GCM)
var decipher = forge.cipher.createDecipher('AES-CBC', key);
decipher.start({iv: iv});
decipher.update(encrypted);
decipher.finish();
// outputs decrypted hex
console.log(decipher.output.toHex());

// encrypt some bytes using GCM mode
var cipher = forge.cipher.createCipher('AES-GCM', key);
cipher.start({
  iv: iv, // should be a 12-byte binary-encoded string or byte buffer
  additionalData: 'binary-encoded string', // optional
  tagLength: 128 // optional, defaults to 128 bits
});
cipher.update(forge.util.createBuffer(someBytes));
cipher.finish();
var encrypted = cipher.output;
var tag = cipher.mode.tag;
// outputs encrypted hex
console.log(encrypted.toHex());
// outputs authentication tag
console.log(tag.toHex());

// decrypt some bytes using GCM mode
var decipher = forge.cipher.createDecipher('AES-GCM', key);
decipher.start({
  iv: iv,
  additionalData: 'binary-encoded string', // optional
  tagLength: 128, // optional, defaults to 128 bits
  tag: tag // authentication tag from encryption
});
decipher.update(encrypted);
var pass = decipher.finish();
// pass is false if there was a failure (eg: authentication tag didn't match)
if(pass) {
  // outputs decrypted hex
  console.log(decipher.output.toHex());
}
```

Using forge in node.js to match openssl's "enc" command line tool (**Note**: OpenSSL "enc" uses a non-standard file format with a custom key derivation function and a fixed iteration count of 1, which some consider less secure than alternatives such as [OpenPGP](https://tools.ietf.org/html/rfc4880)/[GnuPG](https://www.gnupg.org/)):

```js
var forge = require('node-forge');
var fs = require('fs');

// openssl enc -des3 -in input.txt -out input.enc
function encrypt(password) {
  var input = fs.readFileSync('input.txt', {encoding: 'binary'});

  // 3DES key and IV sizes
  var keySize = 24;
  var ivSize = 8;

  // get derived bytes
  // Notes:
  // 1. If using an alternative hash (eg: "-md sha1") pass
  //   "forge.md.sha1.create()" as the final parameter.
  // 2. If using "-nosalt", set salt to null.
  var salt = forge.random.getBytesSync(8);
  // var md = forge.md.sha1.create(); // "-md sha1"
  var derivedBytes = forge.pbe.opensslDeriveBytes(
    password, salt, keySize + ivSize/*, md*/);
  var buffer = forge.util.createBuffer(derivedBytes);
  var key = buffer.getBytes(keySize);
  var iv = buffer.getBytes(ivSize);

  var cipher = forge.cipher.createCipher('3DES-CBC', key);
  cipher.start({iv: iv});
  cipher.update(forge.util.createBuffer(input, 'binary'));
  cipher.finish();

  var output = forge.util.createBuffer();

  // if using a salt, prepend this to the output:
  if(salt !== null) {
    output.putBytes('Salted__'); // (add to match openssl tool output)
    output.putBytes(salt);
  }
  output.putBuffer(cipher.output);

  fs.writeFileSync('input.enc', output.getBytes(), {encoding: 'binary'});
}

// openssl enc -d -des3 -in input.enc -out input.dec.txt
function decrypt(password) {
  var input = fs.readFileSync('input.enc', {encoding: 'binary'});

  // parse salt from input
  input = forge.util.createBuffer(input, 'binary');
  // skip "Salted__" (if known to be present)
  input.getBytes('Salted__'.length);
  // read 8-byte salt
  var salt = input.getBytes(8);

  // Note: if using "-nosalt", skip above parsing and use
  // var salt = null;

  // 3DES key and IV sizes
  var keySize = 24;
  var ivSize = 8;

  var derivedBytes = forge.pbe.opensslDeriveBytes(
    password, salt, keySize + ivSize);
  var buffer = forge.util.createBuffer(derivedBytes);
  var key = buffer.getBytes(keySize);
  var iv = buffer.getBytes(ivSize);

  var decipher = forge.cipher.createDecipher('3DES-CBC', key);
  decipher.start({iv: iv});
  decipher.update(input);
  var result = decipher.finish(); // check 'result' for true/false

  fs.writeFileSync(
    'input.dec.txt', decipher.output.getBytes(), {encoding: 'binary'});
}
```

<a name="aes" />
### AES

Provides [AES][] encryption and decryption in [CBC][], [CFB][], [OFB][],
[CTR][], and [GCM][] modes. See [CIPHER](#cipher) for examples.

<a name="des" />
### DES

Provides [3DES][] and [DES][] encryption and decryption in [ECB][] and
[CBC][] modes. See [CIPHER](#cipher) for examples.

<a name="rc2" />
### RC2

__Examples__

```js
// generate a random key and IV
var key = forge.random.getBytesSync(16);
var iv = forge.random.getBytesSync(8);

// encrypt some bytes
var cipher = forge.rc2.createEncryptionCipher(key);
cipher.start(iv);
cipher.update(forge.util.createBuffer(someBytes));
cipher.finish();
var encrypted = cipher.output;
// outputs encrypted hex
console.log(encrypted.toHex());

// decrypt some bytes
var cipher = forge.rc2.createDecryptionCipher(key);
cipher.start(iv);
cipher.update(encrypted);
cipher.finish();
// outputs decrypted hex
console.log(cipher.output.toHex());
```
---------------------------------------
## PKI

Provides [X.509][] certificate and RSA public and private key encoding,
decoding, encryption/decryption, and signing/verifying.

<a name="rsa" />
### RSA

__Examples__

```js
var rsa = forge.pki.rsa;

// generate an RSA key pair synchronously
var keypair = rsa.generateKeyPair({bits: 2048, e: 0x10001});

// generate an RSA key pair asynchronously (uses web workers if available)
// use workers: -1 to run a fast core estimator to optimize # of workers
rsa.generateKeyPair({bits: 2048, workers: 2}, function(err, keypair) {
  // keypair.privateKey, keypair.publicKey
});

// generate an RSA key pair in steps that attempt to run for a specified period
// of time on the main JS thread
var state = rsa.createKeyPairGenerationState(2048, 0x10001);
var step = function() {
  // run for 100 ms
  if(!rsa.stepKeyPairGenerationState(state, 100)) {
    setTimeout(step, 1);
  }
  else {
    // done, turn off progress indicator, use state.keys
  }
};
// turn on progress indicator, schedule generation to run
setTimeout(step);

// sign data with a private key and output DigestInfo DER-encoded bytes
// (defaults to RSASSA PKCS#1 v1.5)
var md = forge.md.sha1.create();
md.update('sign this', 'utf8');
var signature = privateKey.sign(md);

// verify data with a public key
// (defaults to RSASSA PKCS#1 v1.5)
var verified = publicKey.verify(md.digest().bytes(), signature);

// sign data using RSASSA-PSS where PSS uses a SHA-1 hash, a SHA-1 based
// masking function MGF1, and a 20 byte salt
var md = forge.md.sha1.create();
md.update('sign this', 'utf8');
var pss = forge.pss.create({
  md: forge.md.sha1.create(),
  mgf: forge.mgf.mgf1.create(forge.md.sha1.create()),
  saltLength: 20
  // optionally pass 'prng' with a custom PRNG implementation
  // optionalls pass 'salt' with a forge.util.ByteBuffer w/custom salt
});
var signature = privateKey.sign(md, pss);

// verify RSASSA-PSS signature
var pss = forge.pss.create({
  md: forge.md.sha1.create(),
  mgf: forge.mgf.mgf1.create(forge.md.sha1.create()),
  saltLength: 20
  // optionally pass 'prng' with a custom PRNG implementation
});
var md = forge.md.sha1.create();
md.update('sign this', 'utf8');
publicKey.verify(md.digest().getBytes(), signature, pss);

// encrypt data with a public key (defaults to RSAES PKCS#1 v1.5)
var encrypted = publicKey.encrypt(bytes);

// decrypt data with a private key (defaults to RSAES PKCS#1 v1.5)
var decrypted = privateKey.decrypt(encrypted);

// encrypt data with a public key using RSAES PKCS#1 v1.5
var encrypted = publicKey.encrypt(bytes, 'RSAES-PKCS1-V1_5');

// decrypt data with a private key using RSAES PKCS#1 v1.5
var decrypted = privateKey.decrypt(encrypted, 'RSAES-PKCS1-V1_5');

// encrypt data with a public key using RSAES-OAEP
var encrypted = publicKey.encrypt(bytes, 'RSA-OAEP');

// decrypt data with a private key using RSAES-OAEP
var decrypted = privateKey.decrypt(encrypted, 'RSA-OAEP');

// encrypt data with a public key using RSAES-OAEP/SHA-256
var encrypted = publicKey.encrypt(bytes, 'RSA-OAEP', {
  md: forge.md.sha256.create()
});

// decrypt data with a private key using RSAES-OAEP/SHA-256
var decrypted = privateKey.decrypt(encrypted, 'RSA-OAEP', {
  md: forge.md.sha256.create()
});

// encrypt data with a public key using RSAES-OAEP/SHA-256/MGF1-SHA-1
// compatible with Java's RSA/ECB/OAEPWithSHA-256AndMGF1Padding
var encrypted = publicKey.encrypt(bytes, 'RSA-OAEP', {
  md: forge.md.sha256.create(),
  mgf1: {
    md: forge.md.sha1.create()
  }
});

// decrypt data with a private key using RSAES-OAEP/SHA-256/MGF1-SHA-1
// compatible with Java's RSA/ECB/OAEPWithSHA-256AndMGF1Padding
var decrypted = privateKey.decrypt(encrypted, 'RSA-OAEP', {
  md: forge.md.sha256.create(),
  mgf1: {
    md: forge.md.sha1.create()
  }
});

```

<a name="rsakem" />
### RSA-KEM

__Examples__

```js
// generate an RSA key pair asynchronously (uses web workers if available)
// use workers: -1 to run a fast core estimator to optimize # of workers
forge.rsa.generateKeyPair({bits: 2048, workers: -1}, function(err, keypair) {
  // keypair.privateKey, keypair.publicKey
});

// generate and encapsulate a 16-byte secret key
var kdf1 = new forge.kem.kdf1(forge.md.sha1.create());
var kem = forge.kem.rsa.create(kdf1);
var result = kem.encrypt(keypair.publicKey, 16);
// result has 'encapsulation' and 'key'

// encrypt some bytes
var iv = forge.random.getBytesSync(12);
var someBytes = 'hello world!';
var cipher = forge.cipher.createCipher('AES-GCM', result.key);
cipher.start({iv: iv});
cipher.update(forge.util.createBuffer(someBytes));
cipher.finish();
var encrypted = cipher.output.getBytes();
var tag = cipher.mode.tag.getBytes();

// send 'encrypted', 'iv', 'tag', and result.encapsulation to recipient

// decrypt encapsulated 16-byte secret key
var kdf1 = new forge.kem.kdf1(forge.md.sha1.create());
var kem = forge.kem.rsa.create(kdf1);
var key = kem.decrypt(keypair.privateKey, result.encapsulation, 16);

// decrypt some bytes
var decipher = forge.cipher.createDecipher('AES-GCM', key);
decipher.start({iv: iv, tag: tag});
decipher.update(forge.util.createBuffer(encrypted));
var pass = decipher.finish();
// pass is false if there was a failure (eg: authentication tag didn't match)
if(pass) {
  // outputs 'hello world!'
  console.log(decipher.output.getBytes());
}

```

<a name="x509" />
### X.509

__Examples__

```js
var pki = forge.pki;

// convert a PEM-formatted public key to a Forge public key
var publicKey = pki.publicKeyFromPem(pem);

// convert a Forge public key to PEM-format
var pem = pki.publicKeyToPem(publicKey);

// convert an ASN.1 SubjectPublicKeyInfo to a Forge public key
var publicKey = pki.publicKeyFromAsn1(subjectPublicKeyInfo);

// convert a Forge public key to an ASN.1 SubjectPublicKeyInfo
var subjectPublicKeyInfo = pki.publicKeyToAsn1(publicKey);

// gets a SHA-1 RSAPublicKey fingerprint a byte buffer
pki.getPublicKeyFingerprint(key);

// gets a SHA-1 SubjectPublicKeyInfo fingerprint a byte buffer
pki.getPublicKeyFingerprint(key, {type: 'SubjectPublicKeyInfo'});

// gets a hex-encoded, colon-delimited SHA-1 RSAPublicKey public key fingerprint
pki.getPublicKeyFingerprint(key, {encoding: 'hex', delimiter: ':'});

// gets a hex-encoded, colon-delimited SHA-1 SubjectPublicKeyInfo public key fingerprint
pki.getPublicKeyFingerprint(key, {
  type: 'SubjectPublicKeyInfo',
  encoding: 'hex',
  delimiter: ':'
});

// gets a hex-encoded, colon-delimited MD5 RSAPublicKey public key fingerprint
pki.getPublicKeyFingerprint(key, {
  md: forge.md.md5.create(),
  encoding: 'hex',
  delimiter: ':'
});

// creates a CA store
var caStore = pki.createCaStore([/* PEM-encoded cert */, ...]);

// add a certificate to the CA store
caStore.addCertificate(certObjectOrPemString);

// gets the issuer (its certificate) for the given certificate
var issuerCert = caStore.getIssuer(subjectCert);

// verifies a certificate chain against a CA store
pki.verifyCertificateChain(caStore, chain, customVerifyCallback);

// signs a certificate using the given private key
cert.sign(privateKey);

// signs a certificate using SHA-256 instead of SHA-1
cert.sign(privateKey, forge.md.sha256.create());

// verifies an issued certificate using the certificates public key
var verified = issuer.verify(issued);

// generate a keypair and create an X.509v3 certificate
var keys = pki.rsa.generateKeyPair(2048);
var cert = pki.createCertificate();
cert.publicKey = keys.publicKey;
// alternatively set public key from a csr
//cert.publicKey = csr.publicKey;
cert.serialNumber = '01';
cert.validity.notBefore = new Date();
cert.validity.notAfter = new Date();
cert.validity.notAfter.setFullYear(cert.validity.notBefore.getFullYear() + 1);
var attrs = [{
  name: 'commonName',
  value: 'example.org'
}, {
  name: 'countryName',
  value: 'US'
}, {
  shortName: 'ST',
  value: 'Virginia'
}, {
  name: 'localityName',
  value: 'Blacksburg'
}, {
  name: 'organizationName',
  value: 'Test'
}, {
  shortName: 'OU',
  value: 'Test'
}];
cert.setSubject(attrs);
// alternatively set subject from a csr
//cert.setSubject(csr.subject.attributes);
cert.setIssuer(attrs);
cert.setExtensions([{
  name: 'basicConstraints',
  cA: true
}, {
  name: 'keyUsage',
  keyCertSign: true,
  digitalSignature: true,
  nonRepudiation: true,
  keyEncipherment: true,
  dataEncipherment: true
}, {
  name: 'extKeyUsage',
  serverAuth: true,
  clientAuth: true,
  codeSigning: true,
  emailProtection: true,
  timeStamping: true
}, {
  name: 'nsCertType',
  client: true,
  server: true,
  email: true,
  objsign: true,
  sslCA: true,
  emailCA: true,
  objCA: true
}, {
  name: 'subjectAltName',
  altNames: [{
    type: 6, // URI
    value: 'http://example.org/webid#me'
  }, {
    type: 7, // IP
    ip: '127.0.0.1'
  }]
}, {
  name: 'subjectKeyIdentifier'
}]);
/* alternatively set extensions from a csr
var extensions = csr.getAttribute({name: 'extensionRequest'}).extensions;
// optionally add more extensions
extensions.push.apply(extensions, [{
  name: 'basicConstraints',
  cA: true
}, {
  name: 'keyUsage',
  keyCertSign: true,
  digitalSignature: true,
  nonRepudiation: true,
  keyEncipherment: true,
  dataEncipherment: true
}]);
cert.setExtensions(extensions);
*/
// self-sign certificate
cert.sign(keys.privateKey);

// convert a Forge certificate to PEM
var pem = pki.certificateToPem(cert);

// convert a Forge certificate from PEM
var cert = pki.certificateFromPem(pem);

// convert an ASN.1 X.509x3 object to a Forge certificate
var cert = pki.certificateFromAsn1(obj);

// convert a Forge certificate to an ASN.1 X.509v3 object
var asn1Cert = pki.certificateToAsn1(cert);
```

<a name="pkcs5" />
### PKCS#5

Provides the password-based key-derivation function from [PKCS#5][].

__Examples__

```js
// generate a password-based 16-byte key
// note an optional message digest can be passed as the final parameter
var salt = forge.random.getBytesSync(128);
var derivedKey = forge.pkcs5.pbkdf2('password', salt, numIterations, 16);

// generate key asynchronously
// note an optional message digest can be passed before the callback
forge.pkcs5.pbkdf2('password', salt, numIterations, 16, function(err, derivedKey) {
  // do something w/derivedKey
});
```

<a name="pkcs7" />
### PKCS#7

Provides cryptographically protected messages from [PKCS#7][].

__Examples__

```js
// convert a message from PEM
var p7 = forge.pkcs7.messageFromPem(pem);
// look at p7.recipients

// find a recipient by the issuer of a certificate
var recipient = p7.findRecipient(cert);

// decrypt
p7.decrypt(p7.recipients[0], privateKey);

// create a p7 enveloped message
var p7 = forge.pkcs7.createEnvelopedData();

// add a recipient
var cert = forge.pki.certificateFromPem(certPem);
p7.addRecipient(cert);

// set content
p7.content = forge.util.createBuffer('Hello');

// encrypt
p7.encrypt();

// convert message to PEM
var pem = forge.pkcs7.messageToPem(p7);

// create a degenerate PKCS#7 certificate container
// (CRLs not currently supported, only certificates)
var p7 = forge.pkcs7.createSignedData();
p7.addCertificate(certOrCertPem1);
p7.addCertificate(certOrCertPem2);
var pem = forge.pkcs7.messageToPem(p7);

// create PKCS#7 signed data with authenticatedAttributes
// attributes include: PKCS#9 content-type, message-digest, and signing-time
var p7 = forge.pkcs7.createSignedData();
p7.content = forge.util.createBuffer('Some content to be signed.', 'utf8');
p7.addCertificate(certOrCertPem);
p7.addSigner({
  key: privateKeyAssociatedWithCert,
  certificate: certOrCertPem,
  digestAlgorithm: forge.pki.oids.sha256,
  authenticatedAttributes: [{
    type: forge.pki.oids.contentType,
    value: forge.pki.oids.data
  }, {
    type: forge.pki.oids.messageDigest
    // value will be auto-populated at signing time
  }, {
    type: forge.pki.oids.signingTime,
    // value can also be auto-populated at signing time
    value: new Date()
  }]
});
p7.sign();
var pem = forge.pkcs7.messageToPem(p7);

```

<a name="pkcs8" />
### PKCS#8

__Examples__

```js
var pki = forge.pki;

// convert a PEM-formatted private key to a Forge private key
var privateKey = pki.privateKeyFromPem(pem);

// convert a Forge private key to PEM-format
var pem = pki.privateKeyToPem(privateKey);

// convert an ASN.1 PrivateKeyInfo or RSAPrivateKey to a Forge private key
var privateKey = pki.privateKeyFromAsn1(rsaPrivateKey);

// convert a Forge private key to an ASN.1 RSAPrivateKey
var rsaPrivateKey = pki.privateKeyToAsn1(privateKey);

// wrap an RSAPrivateKey ASN.1 object in a PKCS#8 ASN.1 PrivateKeyInfo
var privateKeyInfo = pki.wrapRsaPrivateKey(rsaPrivateKey);

// convert a PKCS#8 ASN.1 PrivateKeyInfo to PEM
var pem = pki.privateKeyInfoToPem(privateKeyInfo);

// encrypts a PrivateKeyInfo and outputs an EncryptedPrivateKeyInfo
var encryptedPrivateKeyInfo = pki.encryptPrivateKeyInfo(
  privateKeyInfo, 'password', {
    algorithm: 'aes256', // 'aes128', 'aes192', 'aes256', '3des'
  });

// decrypts an ASN.1 EncryptedPrivateKeyInfo
var privateKeyInfo = pki.decryptPrivateKeyInfo(
  encryptedPrivateKeyInfo, 'password');

// converts an EncryptedPrivateKeyInfo to PEM
var pem = pki.encryptedPrivateKeyToPem(encryptedPrivateKeyInfo);

// converts a PEM-encoded EncryptedPrivateKeyInfo to ASN.1 format
var encryptedPrivateKeyInfo = pki.encryptedPrivateKeyFromPem(pem);

// wraps and encrypts a Forge private key and outputs it in PEM format
var pem = pki.encryptRsaPrivateKey(privateKey, 'password');

// encrypts a Forge private key and outputs it in PEM format using OpenSSL's
// proprietary legacy format + encapsulated PEM headers (DEK-Info)
var pem = pki.encryptRsaPrivateKey(privateKey, 'password', {legacy: true});

// decrypts a PEM-formatted, encrypted private key
var privateKey = pki.decryptRsaPrivateKey(pem, 'password');

// sets an RSA public key from a private key
var publicKey = pki.setRsaPublicKey(privateKey.n, privateKey.e);
```

<a name="pkcs10" />
### PKCS#10

Provides certification requests or certificate signing requests (CSR) from
[PKCS#10][].

__Examples__

```js
// generate a key pair
var keys = forge.pki.rsa.generateKeyPair(1024);

// create a certification request (CSR)
var csr = forge.pki.createCertificationRequest();
csr.publicKey = keys.publicKey;
csr.setSubject([{
  name: 'commonName',
  value: 'example.org'
}, {
  name: 'countryName',
  value: 'US'
}, {
  shortName: 'ST',
  value: 'Virginia'
}, {
  name: 'localityName',
  value: 'Blacksburg'
}, {
  name: 'organizationName',
  value: 'Test'
}, {
  shortName: 'OU',
  value: 'Test'
}]);
// set (optional) attributes
csr.setAttributes([{
  name: 'challengePassword',
  value: 'password'
}, {
  name: 'unstructuredName',
  value: 'My Company, Inc.'
}, {
  name: 'extensionRequest',
  extensions: [{
    name: 'subjectAltName',
    altNames: [{
      // 2 is DNS type
      type: 2,
      value: 'test.domain.com'
    }, {
      type: 2,
      value: 'other.domain.com',
    }, {
      type: 2,
      value: 'www.domain.net'
    }]
  }]
}]);

// sign certification request
csr.sign(keys.privateKey);

// verify certification request
var verified = csr.verify();

// convert certification request to PEM-format
var pem = forge.pki.certificationRequestToPem(csr);

// convert a Forge certification request from PEM-format
var csr = forge.pki.certificationRequestFromPem(pem);

// get an attribute
csr.getAttribute({name: 'challengePassword'});

// get extensions array
csr.getAttribute({name: 'extensionRequest'}).extensions;

```

<a name="pkcs12" />
### PKCS#12

Provides the cryptographic archive file format from [PKCS#12][].

__Examples__

```js
// decode p12 from base64
var p12Der = forge.util.decode64(p12b64);
// get p12 as ASN.1 object
var p12Asn1 = forge.asn1.fromDer(p12Der);
// decrypt p12 using the password 'password'
var p12 = forge.pkcs12.pkcs12FromAsn1(p12Asn1, 'password');
// decrypt p12 using non-strict parsing mode (resolves some ASN.1 parse errors)
var p12 = forge.pkcs12.pkcs12FromAsn1(p12Asn1, false, 'password');
// decrypt p12 using literally no password (eg: Mac OS X/apple push)
var p12 = forge.pkcs12.pkcs12FromAsn1(p12Asn1);
// decrypt p12 using an "empty" password (eg: OpenSSL with no password input)
var p12 = forge.pkcs12.pkcs12FromAsn1(p12Asn1, '');
// p12.safeContents is an array of safe contents, each of
// which contains an array of safeBags

// get bags by friendlyName
var bags = p12.getBags({friendlyName: 'test'});
// bags are key'd by attribute type (here "friendlyName")
// and the key values are an array of matching objects
var cert = bags.friendlyName[0];

// get bags by localKeyId
var bags = p12.getBags({localKeyId: buffer});
// bags are key'd by attribute type (here "localKeyId")
// and the key values are an array of matching objects
var cert = bags.localKeyId[0];

// get bags by localKeyId (input in hex)
var bags = p12.getBags({localKeyIdHex: '7b59377ff142d0be4565e9ac3d396c01401cd879'});
// bags are key'd by attribute type (here "localKeyId", *not* "localKeyIdHex")
// and the key values are an array of matching objects
var cert = bags.localKeyId[0];

// get bags by type
var bags = p12.getBags({bagType: forge.pki.oids.certBag});
// bags are key'd by bagType and each bagType key's value
// is an array of matches (in this case, certificate objects)
var cert = bags[forge.pki.oids.certBag][0];

// get bags by friendlyName and filter on bag type
var bags = p12.getBags({
  friendlyName: 'test',
  bagType: forge.pki.oids.certBag
});

// get key bags
var bags = p12.getBags({bagType: forge.pki.oids.keyBag});
// get key
var bag = bags[forge.pki.oids.keyBag][0];
var key = bag.key;
// if the key is in a format unrecognized by forge then
// bag.key will be `null`, use bag.asn1 to get the ASN.1
// representation of the key
if(bag.key === null) {
  var keyAsn1 = bag.asn1;
  // can now convert back to DER/PEM/etc for export
}

// generate a p12 using AES (default)
var p12Asn1 = forge.pkcs12.toPkcs12Asn1(
  privateKey, certificateChain, 'password');

// generate a p12 that can be imported by Chrome/Firefox
// (requires the use of Triple DES instead of AES)
var p12Asn1 = forge.pkcs12.toPkcs12Asn1(
  privateKey, certificateChain, 'password',
  {algorithm: '3des'});

// base64-encode p12
var p12Der = forge.asn1.toDer(p12Asn1).getBytes();
var p12b64 = forge.util.encode64(p12Der);

// create download link for p12
var a = document.createElement('a');
a.download = 'example.p12';
a.setAttribute('href', 'data:application/x-pkcs12;base64,' + p12b64);
a.appendChild(document.createTextNode('Download'));
```

<a name="asn" />
### ASN.1

Provides [ASN.1][] DER encoding and decoding.

__Examples__

```js
var asn1 = forge.asn1;

// create a SubjectPublicKeyInfo
var subjectPublicKeyInfo =
  asn1.create(asn1.Class.UNIVERSAL, asn1.Type.SEQUENCE, true, [
    // AlgorithmIdentifier
    asn1.create(asn1.Class.UNIVERSAL, asn1.Type.SEQUENCE, true, [
      // algorithm
      asn1.create(asn1.Class.UNIVERSAL, asn1.Type.OID, false,
        asn1.oidToDer(pki.oids['rsaEncryption']).getBytes()),
      // parameters (null)
      asn1.create(asn1.Class.UNIVERSAL, asn1.Type.NULL, false, '')
    ]),
    // subjectPublicKey
    asn1.create(asn1.Class.UNIVERSAL, asn1.Type.BITSTRING, false, [
      // RSAPublicKey
      asn1.create(asn1.Class.UNIVERSAL, asn1.Type.SEQUENCE, true, [
        // modulus (n)
        asn1.create(asn1.Class.UNIVERSAL, asn1.Type.INTEGER, false,
          _bnToBytes(key.n)),
        // publicExponent (e)
        asn1.create(asn1.Class.UNIVERSAL, asn1.Type.INTEGER, false,
          _bnToBytes(key.e))
      ])
    ])
  ]);

// serialize an ASN.1 object to DER format
var derBuffer = asn1.toDer(subjectPublicKeyInfo);

// deserialize to an ASN.1 object from a byte buffer filled with DER data
var object = asn1.fromDer(derBuffer);

// convert an OID dot-separated string to a byte buffer
var derOidBuffer = asn1.oidToDer('1.2.840.113549.1.1.5');

// convert a byte buffer with a DER-encoded OID to a dot-separated string
console.log(asn1.derToDer(derOidBuffer));
// output: 1.2.840.113549.1.1.5

// validates that an ASN.1 object matches a particular ASN.1 structure and
// captures data of interest from that structure for easy access
var publicKeyValidator = {
  name: 'SubjectPublicKeyInfo',
  tagClass: asn1.Class.UNIVERSAL,
  type: asn1.Type.SEQUENCE,
  constructed: true,
  captureAsn1: 'subjectPublicKeyInfo',
  value: [{
    name: 'SubjectPublicKeyInfo.AlgorithmIdentifier',
    tagClass: asn1.Class.UNIVERSAL,
    type: asn1.Type.SEQUENCE,
    constructed: true,
    value: [{
      name: 'AlgorithmIdentifier.algorithm',
      tagClass: asn1.Class.UNIVERSAL,
      type: asn1.Type.OID,
      constructed: false,
      capture: 'publicKeyOid'
    }]
  }, {
    // subjectPublicKey
    name: 'SubjectPublicKeyInfo.subjectPublicKey',
    tagClass: asn1.Class.UNIVERSAL,
    type: asn1.Type.BITSTRING,
    constructed: false,
    value: [{
      // RSAPublicKey
      name: 'SubjectPublicKeyInfo.subjectPublicKey.RSAPublicKey',
      tagClass: asn1.Class.UNIVERSAL,
      type: asn1.Type.SEQUENCE,
      constructed: true,
      optional: true,
      captureAsn1: 'rsaPublicKey'
    }]
  }]
};

var capture = {};
var errors = [];
if(!asn1.validate(
  publicKeyValidator, subjectPublicKeyInfo, validator, capture, errors)) {
  throw 'ASN.1 object is not a SubjectPublicKeyInfo.';
}
// capture.subjectPublicKeyInfo contains the full ASN.1 object
// capture.rsaPublicKey contains the full ASN.1 object for the RSA public key
// capture.publicKeyOid only contains the value for the OID
var oid = asn1.derToOid(capture.publicKeyOid);
if(oid !== pki.oids['rsaEncryption']) {
  throw 'Unsupported OID.';
}

// pretty print an ASN.1 object to a string for debugging purposes
asn1.prettyPrint(object);
```

---------------------------------------
## Message Digests

<a name="sha1" />
### SHA1

Provides [SHA-1][] message digests.

__Examples__

```js
var md = forge.md.sha1.create();
md.update('The quick brown fox jumps over the lazy dog');
console.log(md.digest().toHex());
// output: 2fd4e1c67a2d28fced849ee1bb76e7391b93eb12
```

<a name="sha256" />
### SHA256

Provides [SHA-256][] message digests.

__Examples__

```js
var md = forge.md.sha256.create();
md.update('The quick brown fox jumps over the lazy dog');
console.log(md.digest().toHex());
// output: d7a8fbb307d7809469ca9abcb0082e4f8d5651e46d3cdb762d02d0bf37c9e592
```

<a name="sha384" />
### SHA384

Provides [SHA-384][] message digests.

__Examples__

```js
var md = forge.md.sha384.create();
md.update('The quick brown fox jumps over the lazy dog');
console.log(md.digest().toHex());
// output: ca737f1014a48f4c0b6dd43cb177b0afd9e5169367544c494011e3317dbf9a509cb1e5dc1e85a941bbee3d7f2afbc9b1
```

<a name="sha512" />
### SHA512

Provides [SHA-512][] message digests.

__Examples__

```js
// SHA-512
var md = forge.md.sha512.create();
md.update('The quick brown fox jumps over the lazy dog');
console.log(md.digest().toHex());
// output: 07e547d9586f6a73f73fbac0435ed76951218fb7d0c8d788a309d785436bbb642e93a252a954f23912547d1e8a3b5ed6e1bfd7097821233fa0538f3db854fee6

// SHA-512/224
var md = forge.md.sha512.sha224.create();
md.update('The quick brown fox jumps over the lazy dog');
console.log(md.digest().toHex());
// output: 944cd2847fb54558d4775db0485a50003111c8e5daa63fe722c6aa37

// SHA-512/256
var md = forge.md.sha512.sha256.create();
md.update('The quick brown fox jumps over the lazy dog');
console.log(md.digest().toHex());
// output: dd9d67b371519c339ed8dbd25af90e976a1eeefd4ad3d889005e532fc5bef04d
```

<a name="md5" />
### MD5

Provides [MD5][] message digests.

__Examples__

```js
var md = forge.md.md5.create();
md.update('The quick brown fox jumps over the lazy dog');
console.log(md.digest().toHex());
// output: 9e107d9d372bb6826bd81d3542a419d6
```

<a name="hmac" />
### HMAC

Provides [HMAC][] w/any supported message digest algorithm.

__Examples__

```js
var hmac = forge.hmac.create();
hmac.start('sha1', 'Jefe');
hmac.update('what do ya want for nothing?');
console.log(hmac.digest().toHex());
// output: effcdf6ae5eb2fa2d27416d5f184df9c259a7c79
```

---------------------------------------
## Utilities

<a name="prime" />
### Prime

Provides an API for generating large, random, probable primes.

__Examples__

```js
// generate a random prime on the main JS thread
var bits = 1024;
forge.prime.generateProbablePrime(bits, function(err, num) {
  console.log('random prime', num.toString(16));
});

// generate a random prime using Web Workers (if available, otherwise
// falls back to the main thread)
var bits = 1024;
var options = {
  algorithm: {
    name: 'PRIMEINC',
    workers: -1 // auto-optimize # of workers
  }
};
forge.prime.generateProbablePrime(bits, options, function(err, num) {
  console.log('random prime', num.toString(16));
});
```

<a name="prng" />
### PRNG

Provides a [Fortuna][]-based cryptographically-secure pseudo-random number
generator, to be used with a cryptographic function backend, e.g. [AES][]. An
implementation using [AES][] as a backend is provided. An API for collecting
entropy is given, though if window.crypto.getRandomValues is available, it will
be used automatically.

__Examples__

```js
// get some random bytes synchronously
var bytes = forge.random.getBytesSync(32);
console.log(forge.util.bytesToHex(bytes));

// get some random bytes asynchronously
forge.random.getBytes(32, function(err, bytes) {
  console.log(forge.util.bytesToHex(bytes));
});

// collect some entropy if you'd like
forge.random.collect(someRandomBytes);
jQuery().mousemove(function(e) {
  forge.random.collectInt(e.clientX, 16);
  forge.random.collectInt(e.clientY, 16);
});

// specify a seed file for use with the synchronous API if you'd like
forge.random.seedFileSync = function(needed) {
  // get 'needed' number of random bytes from somewhere
  return fetchedRandomBytes;
};

// specify a seed file for use with the asynchronous API if you'd like
forge.random.seedFile = function(needed, callback) {
  // get the 'needed' number of random bytes from somewhere
  callback(null, fetchedRandomBytes);
});

// register the main thread to send entropy or a Web Worker to receive
// entropy on demand from the main thread
forge.random.registerWorker(self);

// generate a new instance of a PRNG with no collected entropy
var myPrng = forge.random.createInstance();
```

<a name="task" />
### Tasks

Provides queuing and synchronizing tasks in a web application.

__Examples__

```js
```

<a name="util" />
### Utilities

Provides utility functions, including byte buffer support, base64,
bytes to/from hex, zlib inflate/deflate, etc.

__Examples__

```js
// encode/decode base64
var encoded = forge.util.encode64(str);
var str = forge.util.decode64(encoded);

// encode/decode UTF-8
var encoded = forge.util.encodeUtf8(str);
var str = forge.util.decodeUtf8(encoded);

// bytes to/from hex
var bytes = forge.util.hexToBytes(hex);
var hex = forge.util.bytesToHex(bytes);

// create an empty byte buffer
var buffer = forge.util.createBuffer();
// create a byte buffer from raw binary bytes
var buffer = forge.util.createBuffer(input, 'raw');
// create a byte buffer from utf8 bytes
var buffer = forge.util.createBuffer(input, 'utf8');

// get the length of the buffer in bytes
buffer.length();
// put bytes into the buffer
buffer.putBytes(bytes);
// put a 32-bit integer into the buffer
buffer.putInt32(10);
// buffer to hex
buffer.toHex();
// get a copy of the bytes in the buffer
bytes.bytes(/* count */);
// empty this buffer and get its contents
bytes.getBytes(/* count */);

// convert a forge buffer into a node.js Buffer
// make sure you specify the encoding as 'binary'
var forgeBuffer = forge.util.createBuffer();
var nodeBuffer = new Buffer(forgeBuffer.getBytes(), 'binary');

// convert a node.js Buffer into a forge buffer
// make sure you specify the encoding as 'binary'
var nodeBuffer = new Buffer();
var forgeBuffer = forge.util.createBuffer(nodeBuffer.toString('binary'));

// parse a URL
var parsed = forge.util.parseUrl('http://example.com/foo?bar=baz');
// parsed.scheme, parsed.host, parsed.port, parsed.path, parsed.fullHost
```

<a name="log" />
### Logging

Provides logging to a javascript console using various categories and
levels of verbosity.

__Examples__

```js
```

<a name="debug" />
### Debugging

Provides storage of debugging information normally inaccessible in
closures for viewing/investigation.

__Examples__

```js
```

<a name="fsp" />
### Flash Socket Policy Module

Provides an [Apache][] module "mod_fsp" that can serve up a Flash Socket
Policy. See `mod_fsp/README` for more details. This module makes it easy to
modify an [Apache][] server to allow cross domain requests to be made to it.


Library Details
---------------

* http://digitalbazaar.com/2010/07/20/javascript-tls-1/
* http://digitalbazaar.com/2010/07/20/javascript-tls-2/

Contact
-------

* Code: https://github.com/digitalbazaar/forge
* Bugs: https://github.com/digitalbazaar/forge/issues
* Email: support@digitalbazaar.com

Donations welcome:

* Donate: paypal@digitalbazaar.com

[AES]: http://en.wikipedia.org/wiki/Advanced_Encryption_Standard
[ASN.1]: http://en.wikipedia.org/wiki/ASN.1
[Apache]: http://httpd.apache.org/
[CFB]: http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation
[CBC]: http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation
[CTR]: http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation
[3DES]: http://en.wikipedia.org/wiki/Triple_DES
[DES]: http://en.wikipedia.org/wiki/Data_Encryption_Standard
[ECB]: http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation
[Fortuna]: http://en.wikipedia.org/wiki/Fortuna_(PRNG)
[GCM]: http://en.wikipedia.org/wiki/GCM_mode
[HMAC]: http://en.wikipedia.org/wiki/HMAC
[JavaScript]: http://en.wikipedia.org/wiki/JavaScript
[MD5]: http://en.wikipedia.org/wiki/MD5
[OFB]: http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation
[PKCS#5]: http://en.wikipedia.org/wiki/PKCS
[PKCS#7]: http://en.wikipedia.org/wiki/Cryptographic_Message_Syntax
[PKCS#10]: http://en.wikipedia.org/wiki/Certificate_signing_request
[PKCS#12]: http://en.wikipedia.org/wiki/PKCS_%E2%99%AF12
[RC2]: http://en.wikipedia.org/wiki/RC2
[SHA-1]: http://en.wikipedia.org/wiki/SHA-1
[SHA-256]: http://en.wikipedia.org/wiki/SHA-256
[SHA-384]: http://en.wikipedia.org/wiki/SHA-384
[SHA-512]: http://en.wikipedia.org/wiki/SHA-512
[TLS]: http://en.wikipedia.org/wiki/Transport_Layer_Security
[X.509]: http://en.wikipedia.org/wiki/X.509
[node.js]: http://nodejs.org/
