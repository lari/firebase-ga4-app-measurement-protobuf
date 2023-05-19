# Proto definition for app-measurement.com/a calls

This is a community-created Proto definition for decoding the Google Analytics (GA4) HTTP requests sent by Firebase Analytics SDK (iOS or Android) to `https://app-measurement.com/a`, `https://region1.app-measurement.com` or other similar domain.

[Protocol Buffers (Protobuf)](https://protobuf.dev) are a mechanism develop by Google for serializing structured data, kind of like XML or JSON.

The [`app-measurement.proto`](./app-measurement.proto) file contains the Protocol buffers definition. A compiled descriptor file, [`app-measurement.desc`](./app-measurement.desc), is also included in the project.

In case you are interested in the process of creating this proto definition, you can read the blog post [Unraveling Firebase Analytics GA4 calls to app-measurement.com](https://larihaataja.com/firebase-ga4-app-measurement-com-calls/).

## Compiling the descriptor

If you want to make changes to the definition and compile the descriptor yourself, you can do so easily.

First, [download & install the protocol buffer compiler](https://github.com/protocolbuffers/protobuf#protocol-compiler-installation). Protobuf support multiple languages such as Python and Java.

Next, compile the `app-measurement.proto` into `app-measurement.desc`:

```bash
$ protoc --descriptor_set_out=app-measurement.desc app-measurement.proto
```

## Usage

Once you have obtain a raw request body to `app-measurement.com/a` you can decode it by first unzipping it and then decoding the Protocol buffers.

### Decode a request body using protocol buffer compiler

Depending on how you get the request body, you might have to unzip it first. Some HTTP proxy software might do this automatically.

```bash
$ gunzip - < request_body_raw.bin > request_body.bin
```

And once you have the unzipped body, you can decode it with the protocol buffer compiler:

```bash
$ protoc --decode=app_measurement.Batch app-measurement.proto < request_body.bin
```

In the above command we basically tell `protoc` to decode the standard input (read from file using `< request_body.bin`) as a message with type `app_measurement.Batch` using definition in `app-measurement.proto` file.

### Decode a request body with Charles HTTP proxy software

[Charles](https://www.charlesproxy.com) — an HTTP proxy software for Windows, Mac and Linux — can decode a request or response body using Protocol buffers. You just need to add the compiled descriptor file `app-measurement.desc` in its descriptor registry and create a viewer mapping.

You can do this by opening `Viewer Mappings...` from the `view` menu at top. Here you can add a mapping to a specific location (e.g. `https://app-measurement.com/a`). You need to add the `app-measurement.desc` to the Descriptor Registry and then select the following settings:

- `Request type`: `Protocol Buffers`
- `Message type`: `app_measurement.Batch`
- `Payload encoding`: `Binary` & `Unknown`

Once the viewer mapping has been added, Charles will automatically decode the request body sent to `app-measurement.com/a`.

## Shortened event, parameter and user property names

Firebase Analytics SDK also shortens some of the default event, event parameter and user property names. Many of these are quite clear abbreviations but not all. Here are lists of abbreviations and their original values.

### Event names:

```
_s = session_start
_e = user_engagement
_vs = screen_view
_ab = app_background ?
_au = app_update ?
```

### Event parameter names:

```
_si  = firebase_screen_id
_et  = engagement_time_msec
_sc  = firebase_screen_class
_sn  = screen name
_o   = firebase_event_origin
_pn  = previous screen name ?
_pc  = previous view controller
_mst = ?
_pv  = previous app version
_pi  = ?
_err = error
_ev  = error parameter
_el  = error code
_r   = realtime
_dbg = ga_debug
```

### User property names:

```
_fi  = first_open_after_install
_fot = first_open_time
_sid = ga_session_id
_sno = ga_session_number
_lte = lifetime_user_engagement (time in ms)
_se  = session_user_engagement (time in ms)
```

## Contributing

This proto definition is not a full match to what Firebase Analytics SDK uses and you might see some keys / values that are not decoded.

You are very welcome to contribute to this project:

- In case you encounter keys / values that are not decoded by this Proto definition, you can create an [issue](https://github.com/lari/firebase-ga4-app-measurement-protobuf/issues) in GitHub and participate in the discussoins so we can unravel the meaning of the particular keys or values.
- If you are able to figure out the meaning of some missing keys / values, please [fork](https://github.com/lari/firebase-ga4-app-measurement-protobuf/fork) this repo and [create a pull request](https://github.com/lari/firebase-ga4-app-measurement-protobuf/pulls) with your changes so that others can benefit from your findings as well.
