# Revision Content

## Issue Reproduction

During the process of trying to persist cookies in the client for testing, I found that when calling the save() method of the persistent-cookiejar jar from [https://github.com/juju/persistent-cookiejar], some cookie parameters were not persisted. After debugging, I discovered that this was because some cookies had a persistent value of false, but their expiration time was set to 9999 years. If these key parameters are missing, it is not possible to simulate a successful login. Therefore, it is necessary to retain these parameters, but the original persistent-cookiejar filters out cookies with a persistent value of false, which prevents them from being saved, resulting in cookies lacking valid information for login.

![image](https://user-images.githubusercontent.com/20622517/231108044-27c3b134-d0b2-4246-830c-fd5ed04e484e.png)

![B3A45DBF622700017521360DD112585E](https://user-images.githubusercontent.com/20622517/231108561-a529f06f-4b82-4914-b84a-9d4b28076991.jpg)

![image](https://user-images.githubusercontent.com/20622517/231108654-d08487a0-4001-47cc-bfe8-998a470ba79b.png)

## Key Code

Below is the original code, which can be seen filtering out non-persistent parameters.
![2AAB7AB0F517716B2AB6898A90EE8141](https://user-images.githubusercontent.com/20622517/231108696-c80c4d6f-07b4-49a9-b308-fbe05a53015a.jpg)

Due to time constraints, I only removed the if condition.
![183F59DDAC3787DC21F5B28AD8CE01D7](https://user-images.githubusercontent.com/20622517/231108724-59b96dbf-c546-477c-9b8a-b247a6363d1f.jpg)


# cookiejar
--
    import "github.com/juju/persistent-cookiejar"

Package cookiejar implements an in-memory RFC 6265-compliant http.CookieJar.

This implementation is a fork of net/http/cookiejar which also implements
methods for dumping the cookies to persistent storage and retrieving them.

## Usage

#### func  DefaultCookieFile

```go
func DefaultCookieFile() string
```
DefaultCookieFile returns the default cookie file to use for persisting cookie
data. The following names will be used in decending order of preference:

    - the value of the $GOCOOKIES environment variable.
    - $HOME/.go-cookies

#### type Jar

```go
type Jar struct {
}
```

Jar implements the http.CookieJar interface from the net/http package.

#### func  New

```go
func New(o *Options) (*Jar, error)
```
New returns a new cookie jar. A nil *Options is equivalent to a zero Options.

New will return an error if the cookies could not be loaded from the file for
any reason than if the file does not exist.

#### func (*Jar) Cookies

```go
func (j *Jar) Cookies(u *url.URL) (cookies []*http.Cookie)
```
Cookies implements the Cookies method of the http.CookieJar interface.

It returns an empty slice if the URL's scheme is not HTTP or HTTPS.

#### func (*Jar) Save

```go
func (j *Jar) Save() error
```
Save saves the cookies to the persistent cookie file. Before the file is
written, it reads any cookies that have been stored from it and merges them into
j.

#### func (*Jar) SetCookies

```go
func (j *Jar) SetCookies(u *url.URL, cookies []*http.Cookie)
```
SetCookies implements the SetCookies method of the http.CookieJar interface.

It does nothing if the URL's scheme is not HTTP or HTTPS.

#### type Options

```go
type Options struct {
	// PublicSuffixList is the public suffix list that determines whether
	// an HTTP server can set a cookie for a domain.
	//
	// If this is nil, the public suffix list implementation in golang.org/x/net/publicsuffix
	// is used.
	PublicSuffixList PublicSuffixList

	// Filename holds the file to use for storage of the cookies.
	// If it is empty, the value of DefaultCookieFile will be used.
	Filename string
}
```

Options are the options for creating a new Jar.

#### type PublicSuffixList

```go
type PublicSuffixList interface {
	// PublicSuffix returns the public suffix of domain.
	//
	// TODO: specify which of the caller and callee is responsible for IP
	// addresses, for leading and trailing dots, for case sensitivity, and
	// for IDN/Punycode.
	PublicSuffix(domain string) string

	// String returns a description of the source of this public suffix
	// list. The description will typically contain something like a time
	// stamp or version number.
	String() string
}
```

PublicSuffixList provides the public suffix of a domain. For example:

    - the public suffix of "example.com" is "com",
    - the public suffix of "foo1.foo2.foo3.co.uk" is "co.uk", and
    - the public suffix of "bar.pvt.k12.ma.us" is "pvt.k12.ma.us".

Implementations of PublicSuffixList must be safe for concurrent use by multiple
goroutines.

An implementation that always returns "" is valid and may be useful for testing
but it is not secure: it means that the HTTP server for foo.com can set a cookie
for bar.com.

A public suffix list implementation is in the package
golang.org/x/net/publicsuffix.
