+++
title = "Getting the requestor's IP address."
date = 2013-05-24T22:27:00.002-05:00
updated = 2013-05-30T18:31:03.315-05:00
draft = false
url = '/2013/05/getting-requestors-ip-address.html'
tags = [".Net","IP Address","Asp.Net"]
+++

Getting the client's IP
=======================

While it seems it should be straightforward to get the IP address of the client making a web request in the ASP.Net pipeline, there are a few surprises lurking.  In my link-click tracking application, I ended up with this snippet based on Grant Burton's [great post](http://www.grantburton.com/2008/11/30/fix-for-incorrect-ip-addresses-in-wordpress-comments/).  
```
namespace Phydeaux.Helpers
{
    using System;
    using System.Net;
    using System.Net.Sockets;
    using System.Web;

    public static class ClientIP
    {
        public static string ClientIPFromRequest(this HttpRequestBase request, bool skipPrivate)
        {
            foreach (var item in s_HeaderItems)
            {
                var ipString = request.Headers[item.Key];

                if (String.IsNullOrEmpty(ipString))
                    continue;

                if (item.Split)
                {
                    foreach (var ip in ipString.Split(','))
                        if (ValidIP(ip, skipPrivate))
                            return ip;
                }
                else
                {
                    if (ValidIP(ipString, skipPrivate))
                        return ipString;
                }
            }

            return request.UserHostAddress;
        }

        private static bool ValidIP(string ip, bool skipPrivate)
        {
            IPAddress ipAddr;

            ip = ip == null ? String.Empty : ip.Trim();

            if (0 == ip.Length
                || false == IPAddress.TryParse(ip, out ipAddr)
                || (ipAddr.AddressFamily != AddressFamily.InterNetwork
                    && ipAddr.AddressFamily != AddressFamily.InterNetworkV6))
                return false;

            if (skipPrivate && ipAddr.AddressFamily == AddressFamily.InterNetwork)
            {
                var addr = IpRange.AddrToUInt64(ipAddr);
                foreach (var range in s_PrivateRanges)
                {
                    if (range.Encompasses(addr))
                        return false;
                }
            }

            return true;
        }

        /// 
        /// Provides a simple class that understands how to parse and
        /// compare IP addresses (IPV4 and IPV6) ranges.
        /// 
        private sealed class IpRange
        {
            private readonly UInt64 _start;
            private readonly UInt64 _end;

            public IpRange(string startStr, string endStr)
            {
                _start = ParseToUInt64(startStr);
                _end = ParseToUInt64(endStr);
            }

            public static UInt64 AddrToUInt64(IPAddress ip)
            {
                var ipBytes = ip.GetAddressBytes();
                UInt64 value = 0;

                foreach (var abyte in ipBytes)
                {
                    value <<= 8;    // shift
                    value += abyte;
                }

                return value;
            }

            public static UInt64 ParseToUInt64(string ipStr)
            {
                var ip = IPAddress.Parse(ipStr);
                return AddrToUInt64(ip);
            }

            public bool Encompasses(UInt64 addrValue)
            {
                return _start <= addrValue && addrValue <= _end;
            }

            public bool Encompasses(IPAddress addr)
            {
                var value = AddrToUInt64(addr);
                return Encompasses(value);
            }
        };

        private static readonly IpRange[] s_PrivateRanges =
            new IpRange[] { 
                    new IpRange("0.0.0.0","2.255.255.255"),
                    new IpRange("10.0.0.0","10.255.255.255"),
                    new IpRange("127.0.0.0","127.255.255.255"),
                    new IpRange("169.254.0.0","169.254.255.255"),
                    new IpRange("172.16.0.0","172.31.255.255"),
                    new IpRange("192.0.2.0","192.0.2.255"),
                    new IpRange("192.168.0.0","192.168.255.255"),
                    new IpRange("255.255.255.0","255.255.255.255")
            };


        /// 
        /// Describes a header item (key) and if it is expected to be 
        /// a comma-delimited string
        /// 
        private sealed class HeaderItem
        {
            public readonly string Key;
            public readonly bool Split;

            public HeaderItem(string key, bool split)
            {
                Key = key;
                Split = split;
            }
        }

        // order is in trust/use order top to bottom
        private static readonly HeaderItem[] s_HeaderItems =
            new HeaderItem[] { 
                    new HeaderItem("HTTP_CLIENT_IP",false),
                    new HeaderItem("HTTP_X_FORWARDED_FOR",true),
                    new HeaderItem("HTTP_X_FORWARDED",false),
                    new HeaderItem("HTTP_X_CLUSTER_CLIENT_IP",false),
                    new HeaderItem("HTTP_FORWARDED_FOR",false),
                    new HeaderItem("HTTP_FORWARDED",false),
                    new HeaderItem("HTTP_VIA",false),
                    new HeaderItem("REMOTE_ADDR",false)
            };
    }
}

```

---

### Comments

#### Hi. The ValidIp() method has an if statement that …

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2013-05-25T12:06:35.567-05:00">May 6, 2013</time>

Hi.  
The ValidIp() method has an if statement that always returns false, which i think is weird.  
  
if (skipPrivate && ipAddr.AddressFamily == AddressFamily.InterNetwork) ....  
  
/J
---

#### skipPrivate is an argument to the public function …

[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2013-05-25T13:10:31.306-05:00">May 6, 2013</time>

skipPrivate is an argument to the public function and is passed-through.
---

#### Yeah, but should't if (range.Encompasses(addr)…

[Anonymous](mailto:noreply@blogger.com) - <time datetime="2013-05-25T13:21:44.625-05:00">May 6, 2013</time>

Yeah, but should't  
if (range.Encompasses(addr))  
return false;  
  
  
  
return true instead?
---

#### No, we're deciding if the IP we're looking…


[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2013-05-25T13:25:00.546-05:00">May 6, 2013</time>

No, we're deciding if the IP we're looking at is **valid**. If it is encompassed by one of the "private IP" ranges, it is **not** valid, so we return immediately.
---

#### There seems to be a bug in the code posted. I th…


[Anonymous](mailto:noreply@blogger.com) - <time datetime="2013-05-30T18:11:33.103-05:00">May 4, 2013</time>

There seems to be a bug in the code posted.  
  
I think:  
  
if (skipPrivate && ipAddr.AddressFamily == AddressFamily.InterNetwork)  
  
should be !skipPrivate  
  
And then outside of the iteration of s\_PrivateRanges there should not be a 'return false;'. That line should be removed. We only want to return false inside the iteration if the supplied IP address falls within the range.
---

#### No…

Please read the entire function and see wh...
[IDisposable](https://www.blogger.com/profile/02275315449689041289 "noreply@blogger.com") - <time datetime="2013-05-30T18:37:47.119-05:00">May 4, 2013</time>

No.  
  
Please read the entire function and see what it is doing.  
  
1\. Action method gets called and wants the IP for a request  
  
2\. Calls _ClientIPFromRequest_ and expects a **valid** IP. **IF** the client wants to skip private IPs, it will pass _true_  
  
3\. For each header item that might have an IP (in order of interest/usefulness) we call _ValidIP_ with the skip flag specified.  
  
4\. _ValidIP_ then decides if the IP is potentially valid; No if blank (so return _false_), No if not a valid IP address string representation (so return _false_), No if not a IPV4 or IPV6 address (so return _false_).  
  
4\. If a IPV4 **and**we've been asked to skip private IPs, then check all the private IP ranges listed in _s\_PrivateRanges_ one at a time. If one of the ranges encompasses the IP address, return _false_  
  
5\. If **not** filtering out private addresses, or the address isn't IPV4, or the address isn't in one of the private ranges, then we return _true_  
  
6\. For the first header that yields a true from _ValidIP_ we return that string.  
  
7\. Failing that, we simply return the _request.UserHostAddress_ that the .Net framework set.
---
