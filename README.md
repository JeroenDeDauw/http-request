# Http request

[![Build Status](https://secure.travis-ci.org/onoi/http-request.svg?branch=master)](http://travis-ci.org/onoi/http-request)
[![Code Coverage](https://scrutinizer-ci.com/g/onoi/http-request/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/onoi/http-request/?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/onoi/http-request/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/onoi/http-request/?branch=master)
[![Latest Stable Version](https://poser.pugx.org/onoi/http-request/version.png)](https://packagist.org/packages/onoi/http-request)
[![Packagist download count](https://poser.pugx.org/onoi/http-request/d/total.png)](https://packagist.org/packages/onoi/http-request)
[![Dependency Status](https://www.versioneye.com/php/onoi:http-request/badge.png)](https://www.versioneye.com/php/onoi:http-request)

A minimalistic http/curl request interface that was part of the [Semantic MediaWiki][smw] code base and
is now being deployed as independent library.

This library provides:

- `HttpRequest` interface
- `CurlRequest` as cURL implementation of the `HttpRequest`
- `CachedCurlRequest` extends `CurlRequest` to support low-level caching on repeated requests
- `AsyncCurlRequest` to make use of the cURL multi stack feature

## Requirements

- PHP 5.3 or later
- lib-curl

## Installation

The recommended installation method for this library is by adding the
dependency to your [composer.json][composer].

```json
{
	"require": {
		"onoi/http-request": "~1.0"
	}
}
```

## Usage

```php
use Onoi\HttpRequest\HttpRequest;
use Onoi\HttpRequest\Exception\BadHttpResponseException;
use Onoi\HttpRequest\Exception\HttpConnectionException;

class Foo {

	private $httpRequest = null;

	public function __constructor( HttpRequest $httpRequest ) {
		$this->httpRequest = $httpRequest;
	}

	public function makeHttpRequestTo( $url ) {

		$this->httpRequest->setOption( CURLOPT_URL, $url );

		if ( !$this->httpRequest->ping() ) {
			throw new HttpConnectionException( "Couldn't connect" );
		}

		$this->httpRequest->setOption( CURLOPT_RETURNTRANSFER, true );

		$this->httpRequest->setOption( CURLOPT_HTTPHEADER, array(
			'Accept: application/x-turtle'
		) );

		$response = $this->httpRequest->execute();

		if ( $this->httpRequest->getLastErrorCode() == 0 ) {
			return $response;
		}

		throw new BadHttpResponseException( $this->httpRequest );
	}
}
```
```php
$httpRequestFactory = new HttpRequestFactory();

$instance = new Foo( $httpRequestFactory->newCurlRequest() );
$instance->makeHttpRequestTo( 'http://example.org' );

OR

$cacheFactory = new CacheFactory();

$compositeCache = $cacheFactory->newCompositeCache( array(
	$cacheFactory->newFixedInMemoryLruCache( 500 ),
	$cacheFactory->newDoctrineCache( new \Doctrine\Common\Cache\RedisCache() )
) );

$httpRequestFactory = new HttpRequestFactory( $compositeCache );

$cachedRequest = $httpRequestFactory->newCachedCurlRequest();
$cachedRequest->setExpiryInSeconds( 60 * 60 );

$instance = new Foo( $cachedRequest );
$instance->makeHttpRequestTo( 'http://example.org' );
```

## Contribution and support

If you want to contribute work to the project please subscribe to the
developers mailing list and have a look at the [contribution guidelinee](/CONTRIBUTING.md). A list of people who have made contributions in the past can be found [here][contributors].

* [File an issue](https://github.com/onoi/http-request/issues)
* [Submit a pull request](https://github.com/onoi/http-request/pulls)

### Tests

The library provides unit tests that covers the core-functionality normally run by the [continues integration platform][travis]. Tests can also be executed manually using the PHPUnit configuration file found in the root directory.

### Release notes

* 1.0.0 initial release (2015-07-22)

## License

[GNU General Public License 2.0 or later][license].

[composer]: https://getcomposer.org/
[contributors]: https://github.com/onoi/http-request/graphs/contributors
[license]: https://www.gnu.org/copyleft/gpl.html
[travis]: https://travis-ci.org/onoi/http-request
[smw]: https://github.com/SemanticMediaWiki/SemanticMediaWiki/
