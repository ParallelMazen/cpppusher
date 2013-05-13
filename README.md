Pusher++ (pusherpp)
===================
A lightweight and thread-safe C++ server library to push message through [Pusher.com](www.pusher.com) service.

Prerequisites
-------------
+ A compiler that supports the directive `-std=c++0x`
+ The following libraries are required:
	- libcurl
	- libcrypto (from OpenSSL)

Installing
----------
+ The usual
	- `./configure`
	- `make`
	- `[sudo] make install`
+ In case you get an error similar to `mv: cannot stat 'pusherpp/include/.deps/SomeFile.Tpo': No such file or directory`, then run `libtoolize -f` before running `./configure`

Linking
-------
+ Make sure you either have `/ust/local/lib` in your `LIBDIR`, or include it in you linker lookup dir (e.g. `-L/usr/local/lib`)
+ To link to the library, use `-lpusherpp`
+ You also need to link to the following libraries:
	- libcurl (`-lcurl`)
	- libcrypto (`-lcrypto`)

Features
--------
+ Blocking calls; adding parallelism is up to you (well, so far)
+ Thread-safe
+ Supports triggering a particular event on one or multiple channels
+ Duplicate messages can be avoided
+ Supports querying application state
+ Calls to Pusher can go through HTTP or HTTPS

Examples
--------
### Simple Example
```C++
#include <iostream>
#include <pusherpp/CPusher.hpp>
#include <pusherpp/CPusherReply.hpp>
#include <string>
#include <vector>

int main(int argc, char** argv)
{
	bool useSecureHttp = true; // You can specify whether to use HTTPS to communicate with pusher or not
	
	Pusherpp::CPusher pusher("YOUR_APP_ID", "YOUR_KEY", "YOUR_SECRET", useSecureHttp);
	Pusherpp::CPusherReply response; // To store response received from Pusher
	
	// Note that all calls within the library are blocking

	// Get info about a channel -- note that subscription_count is not enabled by default
	response = pusher.getChannelInfo("test_channel", Pusherpp::CPusher::CH_INFO_SUBS_COUNT);
	std::cout << response.message << std::endl;

	// Get info about a presence channel
	response = pusher.getChannelInfo("presence-ghost",
			  Pusherpp::CPusher::CH_INFO_SUBS_COUNT | Pusherpp::CPusher::CH_INFO_USERCOUNT);
	std::cout << response.message << std::endl;

	// Get a list of occupied channels
	response = pusher.getChannels();
	std::cout << response.message << std::endl;

	// Get a list of occupied channels, filtered by prefix
	response = pusher.getChannels("test");
	std::cout << response.message << std::endl;

	// Get count of users in all presence channels
	response = pusher.getChannels("presence-", Pusherpp::CPusher::CH_INFO_USERCOUNT);
	std::cout << response.message << std::endl;

	// Trigger an event on a single channel
	response = pusher.trigger("test_channel", "my_event", "Stuff");
	std::cout << response << std::endl; // You may also output a CPusherReply object. Debug-friendly.

	// Trigger an event on a set of channels
	response = pusher.trigger(std::vector<std::string>({"test_channel", "test_channel2"}),
		"my_event", "Lots of Stuff");
	std::cout << response << std::endl;

	// Trigger an event on a set of channels, excluding a socket_id
	response = pusher.trigger(std::vector<std::string>({"test_channel", "test_channel2"}),
		"my_event", "Lots of Stuff", "28716.7338");
	std::cout << response << std::endl;

	return 0;
}
```

### Multithreaded Example
```C++
#include <iostream>
#include <string>
#include <cstdlib>
#include <thread>
#include <pusherpp/CPusher.hpp>

void threadWork(const Pusherpp::CPusher& pusha, const std::string& msg, int tid)
{
	std::cout << "Thread #" << tid << " | Pushing..." << std::endl;
	std::cout << "Thread #" << tid << " | " << "Message from server: " <<
			  pusha.trigger("test_channel", "my_event", msg) << std::endl;
}

int main(int argc, char** argv)
{
	// NOTE: you need to link to pthread (-lpthread) before using this example

	Pusherpp::CPusher pusher("YOUR APP ID", "YOUR KEY", "YOUR SECRET");

	std::vector<std::thread> tlist;

	// Spawn some threads, each one will post a message to pusher
	for (int i = 0; i < 10; i++)
	{
		tlist.emplace_back(std::thread(threadWork, pusher, std::to_string(i), i));
	}

	// Do something here..
	// .. another thing here..
	// .. search for the Higgs Boson..

	// Wait for all threads to complete
	for (auto& t : tlist)
	{
		t.join();
	}

	return 0;
}

```

TODO
----
- [x] Enabling HTTPS connections to Pusher
- [x] Avoiding duplicates while sending events
- [ ] Support logging
- [ ] Support authentication
- [ ] Support async calls

Changelog
---------
+ May 13, 2013
	- Trivos CI
+ May 10, 2013
	- Added `getChannels()` method to query all channels state
	- Added `getChannelInfo()` method to query status of a single channel
	- Ability to exclude a particular `socket_id` when calling trigger
	- HTTPS connection to pusher enabled
+ May 8, 2013
	- `sendMessage()` is now deprecated, use `trigger()` instead
+ May 7, 2013
	- Error messages reported from Pusher are stored
+ May 5, 2013
	- Now supports pushing to multiple channels
+ May 3, 2013
	- Autoconf enabled, now you  can `configure`/`make`/`make install`
	- Changed to blocking calls
+ May 1, 2013
	- Created
	
License
-------
The MIT License (MIT)

Copyright (c) 2013 Futures Business Development

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
