# Reporting an error with a library or integration

## Is your issue with the library itself or the database?

### [Library](./library.md)
### [Database](./db.md)

## How can I tell?

### Try to reproduce the issue without the library

Often the easiest way to identify if a problem is with a library or the service
is to try to make the same request against the service's HTTP API without using
the library.

For example, say you are seeing a `SocketTimeoutException` from java-cloudant
when getting a document. Try to get the same document using `curl`. If that also
fails then it is likely a database problem.

Generally, if a problem occurs independently of the tooling used to connect to
the database server then the problem is almost certainly with the database
server, not the library or tooling being used.

### HTTP response status codes

The server HTTP response status code can give a good indication of where a
problem is.

#### `4xx`

These status codes indicate a client side problem, but they usually don't mean a
bug in the library being used.

* `400` This could be indicative of a bug in a library since it has sent a bad
request to the server, but it could also be a problem with the arguments you have supplied to a library function. Consider double checking them.
* `401` `403` Are you credentials correct? Have you changed a password recently?
Do you have permission to access the resource you are trying to? Do you need to
authenticate with a proxy as well?
* `404` Is the database or document you are trying to use still there?
* `413` Are you uploading a document or attachment that is too large? If you are sending a bulk request consider breaking it apart into smaller requests.
* `429` You've made too many requests in a given time period. You can configure
many of the libraries to back-off and retry, but ultimately you need a plan that
is large enough for your workload.

#### `5xx`

These status codes indicate a server side problem, but not always with the
database, it could also be a problem with a proxy. Sometimes retrying requests
that have failed with a `5xx` code will succeed. If you don't have a proxy you
can consider these database server problems and you should contact your
[database support](./db.md). If you are using a proxy you should check the
status code and proxy logs to isolate the problem to the database.

### Common problems

* It used to work and now it does not
    * Have you upgraded the version of library you are using? Have you checked
    the changelog or release notes? It is possible the library has a behaviour
    change. Alternatively it could be a [regression bug](./library.md).
    * If you have not changed the library version then something could have
    changed on the server. Contact your [database support](./db.md).

### Client side known issues

#### Connection error masks 413 response

Many of the underlying HTTP libraries used in the integrations do not comply
with [RFC 2616; Section 8.2.2](https://tools.ietf.org/html/rfc2616#section-8.2.2)
and as a result may show a connection error or timeout rather than the correct
`413` response from the server on a HTTP request that exceeds the allowed limits:
* https://github.com/cloudant/java-cloudant/issues/317
* https://github.com/cloudant/python-cloudant/issues/331
