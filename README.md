# Battle Cats Mailbox Private Server

A private server for the mailbox in The Battle Cats.

It allows you to get any cat, talent orb, or item amount you want without
the ban risk of other methods.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/fieryhenry)

## Credits

- [jamesiotio's CITM](https://github.com/jamestiotio/CITM) for the original
    hacking method and the format of the presents. It no longer works due to
    PONOS adding a signature to the server responses.

## Prerequisites

I will not go into detail on how to do all of the individual steps but you can
find tutorials online.

1. You will need to modify the game to always verify the
nyanko-signature of the server responses as we don't know PONOS's private key.
This can be done using [Frida](https://frida.re/) and this script:

```js
let func_name = "_ZN5Botan11PK_Verifier14verify_messageEPKhjS2_j"
// Botan::PK_Verifier::verify_message(...)

Interceptor.attach(Module.findExportByName("libnative-lib.so", func_name), {
    onLeave: function (retval) {
        retval.replace(0x1)
    }
})
```

You also need to replace the <https://nyanko-items.ponosgames.com> URL in the
libnative-lib.so files with your own server URL. This can be done by extracting
the APK using something like
[apktool](https://ibotpeaches.github.io/Apktool/). Then you can modify the
`libnative-lib.so` file using a hex editor (or notepad maybe). Then you can
repack the APK using apktool and sign it using
[apksigner](https://developer.android.com/studio/command-line/apksigner).

The URL needs to be the same length as the original URL and it needs to have
/items/ at the end with underscores padding the rest of the URL.
The underscores are used to make the URL the same length as the original URL.
The URL also needs to start with https and so you can use a service like
[Serveo](https://serveo.net/) to do the https part for you. Just follow the
instructions on the website. I recommend you to get a custom subdomain so it
doesn't change (you can use any subdomain as long as the total url length is
shorter than the original URL).

Example:
Run a command like: `ssh -R myserver:80:localhost:80 serveo.net`
You may need to setup ssh keys for this to work.
This makes your url: `https://myserver.serveo.net` and then you would replace the
ponos url with `https://myserver.serveo.net/items/_`.

The new modding tool will make this process much easier when it is released.

Instead of using a server, you might be able to use something like
[mitmproxy](https://mitmproxy.org/) or [Fiddler](https://www.telerik.com/fiddler)
to modify the server responses. This did not work for me but it might work for
you.

## Installation

1. Install [Python](https://www.python.org/downloads/) >= 3.9
1. Run `pip install -U bcmbps`

## Usage

1. Run `python -m bcmbps` or `py -m bcmbps` depending on
    your system.
1. Run `python -m bcmbps --help` for more information.
1. Read [jamesiotio's CITM](https://github.com/jamestiotio/CITM) on how to
    format the list of presents.
1. Create a file called `presents.json` somewhere and put your presents in it.
1. Run `python -m bcmbps --presents path/to/presents.json` to start
    server.

Example Presents:

```json
[
    {
        "id": 1,
        "title": "Items",
        "body": "Test Body",
        "createdAt": 1688648392,
        "items": [
            {
                "itemId": 22,
                "itemCategory": 0,
                "amount": 50,
                "title": "Catfood"
            },
            {
                "itemId": 29,
                "itemCategory": 0,
                "amount": 5,
                "title": "Platinum Ticket"
            }
        ]
    }
]
```

Note that `created` is now `createdAt`. You also don't need to fill in `country`
or `clientVersion` or `accountId` as the server will do that for you.

## Alternative Method

You can also change the public key the game uses to verify the server responses
to your own public key from a key pair you generated.

You can modify the `nyanko-service-prd.pem` file in the APK to use your own
public key. You can generate a key pair using openssl:

```sh
openssl genrsa -out private.pem 4096
openssl rsa -in private.pem -pubout -out public.pem
```

The problem with this method is that you now need to intercept or use a private
server for every single request that uses the public key as now the game will
reject the official server responses from PONOS.

However, I did still manage to do all of that, but the game refused to upload
the save data to the game servers. It starts the request to an aws url but it
aborts immediately after. I couldn't have changed the url of the server it
uploads to to my own server because the url is from a response of another request with a signature that I don't know how to generate. I gave up on this method after that.

Another problem is that if you have a request with more than one slash in the
same place, serveo.net will respond with a 301 Moved Permanently and redirect to
the url with only one slash. But if the original request was a POST request,
the method will change to GET and break the request. If it instead used a 308
Permanent Redirect then it would keep the method as POST and it would work.
I don't know how to fix this.

I know that this is not a Flask issue because Flask responds correctly with a
308 Permanent Redirect. I think it is a problem with serveo.net.

It's probably possible to do this method but I can't be bothered to do it.

Even though this alternative method is not effective, I spent a lot of time on it so I
wanted to include it here.