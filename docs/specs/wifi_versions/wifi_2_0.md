---
title: 'WiFi Specification v2.0'
permalink: /wifi_2_0
classes: spec
---


# Overview

<p>
The GoPro API allows developers to create apps and utilities that interact with and control a GoPro camera.
</p>

## What can you do with GoPro API?

<p>
The GoPro API allows you to control and query the camera:
<ul>
<li>Capture photo/video media</li>
<li>Get media list</li>
<li>Change settings</li>
<li>Set date/time</li>
<li>Get camera status</li>
<li>Get media metadata (file size, width, height, duration, tags, etc)</li>
<li>and more!</li>
</ul>
</p>


# Supported Cameras

<p>
Below is a table of cameras that support GoPro's public REST API:
</p>

<table border="1">
  <tbody>
    <tr style="background-color: rgb(0,0,0); color: rgb(255,255,255);">
      <td>ID</td>
      <td>Model</td>
      <td>Marketing Name</td>
      <td>Minimal Firmware Version</td>
    </tr>
    <tr>
      <td>57</td>
      <td>H21.01</td>
      <td>HERO10 Black</td>
      <td><a href="https://gopro.com/en/us/update/hero10-black">v01.10.00</a></td>
    </tr>
  </tbody>
</table>


# The Basics


## Turning on Camera WiFi Access Point

<p>
In order to maximize the battery life of the camera, the camera's WiFi AP is turned off by default.
Turning on the WiFi AP requires connecting to the camera via
<a href="https://gopro.github.io/OpenGoPro/ble">Bluetooth Low Energy (BLE)</a>
and sending an AP Control command.
</p>


## Authentication
<p>
Once the WiFi Access Point has been turned on, authentication with the camera simply requries connecting with the
correct SSID and password. This information is available in the camera UI by putting the camera into
<a href="https://community.gopro.com/t5/en/Pair-Your-Camera-with-the-GoPro-App/ta-p/394285">pairing mode</a>
and tapping the "i" in the top-right corner of the screen.
</p>

<p>
Additionally, when the camera is in pairing mode, the SSID and password can be read directly via Bluetooth Low Energy.
See <a href="https://gopro.github.io/OpenGoPro/ble">Services and Characteristics</a> in BLE documentation for details.
</p>


## Request and Response Formats
<p>
The camera will respond to REST commands and queries according to the table below.
Most commands are sent via HTTP/GET and require no special HTTP headers.
Responses come in two parts: The standard HTTP return codes and JSON containing any additional information.
Typically, when the camera accepts a command and begins to (asynchronously) work on it, it will return HTTP 200 (OK)
and empty JSON (i.e. <b>{ }</b>) to indicate success.
If an error occurs, the camera will return a standard HTTP error code and JSON with helpful error/debug information.
</p>

<table border="1">
  <tbody>
    <tr style="background-color: rgb(0,0,0); color: rgb(255,255,255);">
      <td>Protocol</td>
      <td>Address</td>
      <td>Port</td>
      <td>Base URL</td>
    </tr>
    <tr>
      <td>WiFi</td>
      <td>10.5.5.9</td>
      <td>8080</td>
      <td>http://10.5.5.9:8080</td>
    </tr>
  </tbody>
</table>


<p>
Depending on the command sent, the camera can return JSON, binary, or Protobuf data.
<ul>
<li>Get Camera State -> JSON</li>
<li>Get Media Info -> JSON</li>
<li>Get Media <a href="https://gopro.github.io/gpmf-parser">GPMF</a> -> Binary</li>
<li>Get Media List -> JSON</li>
<li>Get Media Screennail (JPEG) -> Binary</li>
<li>Get Media Thumbnail (JPEG) -> Binary</li>
<li>Get Presets -> JSON</li>
</ul>
</p>


## Sending Commands

<p>
Depending on the camera's state, it may not be ready to accept some commands.
This ready state is dependent on the <b>System Busy</b> and the <b>Encoding Active</b> status flags. For example:

<ul>
<li>System Busy flag is set while loading presets, changing settings, formatting sdcard, ...</li>
<li>Encoding Active flag is set while capturing photo/video media</li>
</ul>
</p>

<p>
If the system is not ready, it should reject an incoming command; however, best practice is to always wait for the
System Busy and Encode Active flags to go down before sending messages other than queries to get camera status.
For details regarding camera state, see <a href="#camera-status-codes">Camera Status Codes</a>.
</p>


## Presets
<p>
The camera organizes modes of operation into presets.
A preset is a logical wrapper around a specific camera flatmode and a collection of settings that target different ways of capturing media.
</p>

<p>
The set of presets available to load at any moment depends on the value of certain camera settings, which are outlined in the table below.
</p>

<table border="1">
  <tbody>
    <tr style="background-color: rgb(0,0,0); color: rgb(255,255,255);">
      <td>Setting</td>
      <td>Preset</td>
      <td>Preset ID</td>
    </tr>
    <tr>
      <td rowspan="12">Video Performance Mode: Maximum Video Performance</td>
      <td>Standard</td>
      <td>0x00000000</td>
    </tr>
    <tr>
      <td>Activity</td>
      <td>0x00000001</td>
    </tr>
    <tr>
      <td>Cinematic</td>
      <td>0x00000002</td>
    </tr>
    <tr>
      <td>Ultra Slo-Mo</td>
      <td>0x00000004</td>
    </tr>
    <tr>
      <td>Basic</td>
      <td>0x00000005</td>
    </tr>
    <tr>
      <td>Photo</td>
      <td>0x00010000</td>
    </tr>
    <tr>
      <td>Live Burst</td>
      <td>0x00010001</td>
    </tr>
    <tr>
      <td>Burst Photo</td>
      <td>0x00010002</td>
    </tr>
    <tr>
      <td>Night Photo</td>
      <td>0x00010003</td>
    </tr>
    <tr>
      <td>Time Warp</td>
      <td>0x00020000</td>
    </tr>
    <tr>
      <td>Time Lapse</td>
      <td>0x00020001</td>
    </tr>
    <tr>
      <td>Night Lapse</td>
      <td>0x00020002</td>
    </tr>
    <tr>
      <td rowspan="11">Video Performance Mode: Extended Battery</td>
      <td>Photo</td>
      <td>0x00010000</td>
    </tr>
    <tr>
      <td>Live Burst</td>
      <td>0x00010001</td>
    </tr>
    <tr>
      <td>Burst Photo</td>
      <td>0x00010002</td>
    </tr>
    <tr>
      <td>Night Photo</td>
      <td>0x00010003</td>
    </tr>
    <tr>
      <td>Time Warp</td>
      <td>0x00020000</td>
    </tr>
    <tr>
      <td>Time Lapse</td>
      <td>0x00020001</td>
    </tr>
    <tr>
      <td>Night Lapse</td>
      <td>0x00020002</td>
    </tr>
    <tr>
      <td>Standard [EB]</td>
      <td>0x00080000</td>
    </tr>
    <tr>
      <td>Activity [EB]</td>
      <td>0x00080001</td>
    </tr>
    <tr>
      <td>Cinematic [EB]</td>
      <td>0x00080002</td>
    </tr>
    <tr>
      <td>Slo-Mo [EB]</td>
      <td>0x00080003</td>
    </tr>
    <tr>
      <td rowspan="9">Video Performance Mode: Tripod / Stationary Video</td>
      <td>Photo</td>
      <td>0x00010000</td>
    </tr>
    <tr>
      <td>Live Burst</td>
      <td>0x00010001</td>
    </tr>
    <tr>
      <td>Burst Photo</td>
      <td>0x00010002</td>
    </tr>
    <tr>
      <td>Night Photo</td>
      <td>0x00010003</td>
    </tr>
    <tr>
      <td>Time Warp</td>
      <td>0x00020000</td>
    </tr>
    <tr>
      <td>Time Lapse</td>
      <td>0x00020001</td>
    </tr>
    <tr>
      <td>Night Lapse</td>
      <td>0x00020002</td>
    </tr>
    <tr>
      <td>4K Tripod</td>
      <td>0x00090000</td>
    </tr>
    <tr>
      <td>5.3K Tripod</td>
      <td>0x00090001</td>
    </tr>
  </tbody>
</table>

## Limitations

### HERO10 Black
<ul>
<li>The camera will reject requests to change settings while encoding; for example, if Hindsight feature is active, the user cannot change settings</li>
</ul>

### General

<ul>
<li>Unless changed by the user, GoPro cameras will automatically power off after some time (e.g. 5min, 15min, 30min). The Auto Power Down watchdog timer can be reset by sending periodic keep-alive messages to the camera. It is recommended to send a keep-alive at least once every 120 seconds.</li>
<li>In general, querying the value for a setting that is not associated with the current preset/flatmode results in an undefined value. For example, the user should not try to query the current Photo Digital Lenses (FOV) value while in Standard preset (Video flatmode).</li>
</ul>


# Commands
Using the Open GoPro API, a client can perform various command, control, and query operations!

## Commands Quick Reference
<p>
Below is a table of commands that can be sent to the camera and how to send them.<br />
<span style="color:green">✔</span> Indicates support for all Open GoPro firmware versions.<br />
<span style="color:red">❌</span> Indicates a lack of support for all Open GoPro firmware versions.<br />
>= vXX.YY.ZZ indicates support for firmware versions equal to or newer than vXX.YY.ZZ
</p>

<table border="1">
  <tbody>
    <tr style="background-color: rgb(0,0,0); color: rgb(255,255,255);">
      <td>Command</td>
      <td>Description</td>
      <td>HTTP Method</td>
      <td>Endpoint</td>
      <td>HERO10 Black</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Camera: Digital Zoom</td>
      <td>Digital zoom 50%</td>
      <td>GET</td>
      <td>/gopro/camera/digital_zoom?percent=50</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Camera: Get State</td>
      <td>Get camera state (status + settings)</td>
      <td>GET</td>
      <td>/gopro/camera/state</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Keep-alive</td>
      <td>Send keep-alive</td>
      <td>GET</td>
      <td>/gopro/camera/keep_alive</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Media: GPMF</td>
      <td>Get GPMF data (JPG)</td>
      <td>GET</td>
      <td>/gopro/media/gpmf?path=100GOPRO/XXX.JPG</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Media: GPMF</td>
      <td>Get GPMF data (MP4)</td>
      <td>GET</td>
      <td>/gopro/media/gpmf?path=100GOPRO/XXX.MP4</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Media: Info</td>
      <td>Get media info (JPG)</td>
      <td>GET</td>
      <td>/gopro/media/info?path=100GOPRO/XXX.JPG</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Media: Info</td>
      <td>Get media info (MP4)</td>
      <td>GET</td>
      <td>/gopro/media/info?path=100GOPRO/XXX.MP4</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Media: List</td>
      <td>Get media list</td>
      <td>GET</td>
      <td>/gopro/media/list</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Media: Screennail</td>
      <td>Get screennail for "100GOPRO/xxx.JPG"</td>
      <td>GET</td>
      <td>/gopro/media/screennail?path=100GOPRO/XXX.JPG</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Media: Screennail</td>
      <td>Get screennail for "100GOPRO/xxx.MP4"</td>
      <td>GET</td>
      <td>/gopro/media/screennail?path=100GOPRO/XXX.MP4</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Media: Telemetry</td>
      <td>Get telemetry track data (JPG)</td>
      <td>GET</td>
      <td>/gopro/media/telemetry?path=100GOPRO/XXX.JPG</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Media: Telemetry</td>
      <td>Get telemetry track data (MP4)</td>
      <td>GET</td>
      <td>/gopro/media/telemetry?path=100GOPRO/XXX.MP4</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Media: Thumbnail</td>
      <td>Get thumbnail for "100GOPRO/xxx.JPG"</td>
      <td>GET</td>
      <td>/gopro/media/thumbnail?path=100GOPRO/XXX.JPG</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Media: Thumbnail</td>
      <td>Get thumbnail for "100GOPRO/xxx.MP4"</td>
      <td>GET</td>
      <td>/gopro/media/thumbnail?path=100GOPRO/XXX.MP4</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Media: Turbo Transfer</td>
      <td>Turbo transfer: off</td>
      <td>GET</td>
      <td>/gopro/media/turbo_transfer?p=0</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Media: Turbo Transfer</td>
      <td>Turbo transfer: on</td>
      <td>GET</td>
      <td>/gopro/media/turbo_transfer?p=1</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Open GoPro</td>
      <td>Get version</td>
      <td>GET</td>
      <td>/gopro/version</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Presets: Get Status</td>
      <td>Get preset status</td>
      <td>GET</td>
      <td>/gopro/camera/presets/get</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Standard</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=0</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Activity</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=1</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Cinematic</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=2</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Ultra Slo-Mo</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=4</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Basic</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=5</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Photo</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=65536</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Live Burst</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=65537</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Burst Photo</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=65538</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Night Photo</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=65539</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Time Warp</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=131072</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Time Lapse</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=131073</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Night Lapse</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=131074</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Standard [EB]</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=524288</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Activity [EB]</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=524289</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Cinematic [EB]</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=524290</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>Slo-Mo [EB]</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=524291</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>4K Tripod</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=589824</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Presets: Load</td>
      <td>5.3K Tripod</td>
      <td>GET</td>
      <td>/gopro/camera/presets/load?id=589825</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Presets: Load Group</td>
      <td>Video</td>
      <td>GET</td>
      <td>/gopro/camera/presets/set_group?id=1000</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Presets: Load Group</td>
      <td>Photo</td>
      <td>GET</td>
      <td>/gopro/camera/presets/set_group?id=1001</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Presets: Load Group</td>
      <td>Timelapse</td>
      <td>GET</td>
      <td>/gopro/camera/presets/set_group?id=1002</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Set shutter</td>
      <td>Shutter: on</td>
      <td>GET</td>
      <td>/gopro/camera/shutter/start</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Set shutter</td>
      <td>Shutter: off</td>
      <td>GET</td>
      <td>/gopro/camera/shutter/stop</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>Stream: Start</td>
      <td>Start preview stream</td>
      <td>GET</td>
      <td>/gopro/camera/stream/start</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>Stream: Stop</td>
      <td>Stop preview stream</td>
      <td>GET</td>
      <td>/gopro/camera/stream/stop</td>
      <td><span style="color:green">✔</span></td>
    </tr>
  </tbody>
</table>


# Settings
<p>
GoPro cameras have hundreds of setting options to choose from, all of which can be set using a single endpoint.
The endpoint is configured with a <b>setting id</b> and an <b>option value</b>.
Note that setting option values are not globally unique.
While most option values are enumerated values, some are complex bitmasked values.
</p>

## Settings Quick Reference
<p>
Below is a table of setting options detailing how to set every option supported by Open GoPro cameras.<br />
<span style="color:green">✔</span> Indicates support for all Open GoPro firmware versions.<br />
<span style="color:red">❌</span> Indicates a lack of support for all Open GoPro firmware versions.<br />
>= vXX.YY.ZZ indicates support for firmware versions equal to or newer than vXX.YY.ZZ
</p>

<table border="1">
  <tbody>
    <tr style="background-color: rgb(0,0,0); color: rgb(255,255,255);">
      <td>Setting ID</td>
      <td>Setting</td>
      <td>Option</td>
      <td>HTTP Method</td>
      <td>Endpoint</td>
      <td>HERO10 Black</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>2</td>
      <td>Resolution</td>
      <td>Set video resolution (id: 2) to 4k (id: 1)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=2&option=1</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>2</td>
      <td>Resolution</td>
      <td>Set video resolution (id: 2) to 2.7k (id: 4)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=2&option=4</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>2</td>
      <td>Resolution</td>
      <td>Set video resolution (id: 2) to 2.7k 4:3 (id: 6)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=2&option=6</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>2</td>
      <td>Resolution</td>
      <td>Set video resolution (id: 2) to 1080 (id: 9)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=2&option=9</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>2</td>
      <td>Resolution</td>
      <td>Set video resolution (id: 2) to 4k 4:3 (id: 18)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=2&option=18</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>2</td>
      <td>Resolution</td>
      <td>Set video resolution (id: 2) to 5k 4:3 (id: 25)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=2&option=25</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>2</td>
      <td>Resolution</td>
      <td>Set video resolution (id: 2) to 5.3k (id: 100)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=2&option=100</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>3</td>
      <td>Frames Per Second</td>
      <td>Set video fps (id: 3) to 240 (id: 0)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=3&option=0</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>3</td>
      <td>Frames Per Second</td>
      <td>Set video fps (id: 3) to 120 (id: 1)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=3&option=1</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>3</td>
      <td>Frames Per Second</td>
      <td>Set video fps (id: 3) to 100 (id: 2)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=3&option=2</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>3</td>
      <td>Frames Per Second</td>
      <td>Set video fps (id: 3) to 60 (id: 5)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=3&option=5</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>3</td>
      <td>Frames Per Second</td>
      <td>Set video fps (id: 3) to 50 (id: 6)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=3&option=6</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>3</td>
      <td>Frames Per Second</td>
      <td>Set video fps (id: 3) to 30 (id: 8)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=3&option=8</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>3</td>
      <td>Frames Per Second</td>
      <td>Set video fps (id: 3) to 25 (id: 9)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=3&option=9</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>3</td>
      <td>Frames Per Second</td>
      <td>Set video fps (id: 3) to 24 (id: 10)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=3&option=10</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>3</td>
      <td>Frames Per Second</td>
      <td>Set video fps (id: 3) to 200 (id: 13)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=3&option=13</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>59</td>
      <td>Auto Power Down</td>
      <td>Set auto power down (id: 59) to never (id: 0)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=59&option=0</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>59</td>
      <td>Auto Power Down</td>
      <td>Set auto power down (id: 59) to 5 min (id: 4)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=59&option=4</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>59</td>
      <td>Auto Power Down</td>
      <td>Set auto power down (id: 59) to 15 min (id: 6)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=59&option=6</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>59</td>
      <td>Auto Power Down</td>
      <td>Set auto power down (id: 59) to 30 min (id: 7)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=59&option=7</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>121</td>
      <td>Video Digital Lenses</td>
      <td>Set video digital lenses (id: 121) to wide (id: 0)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=121&option=0</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>121</td>
      <td>Video Digital Lenses</td>
      <td>Set video digital lenses (id: 121) to narrow (id: 2)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=121&option=2</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>121</td>
      <td>Video Digital Lenses</td>
      <td>Set video digital lenses (id: 121) to superview (id: 3)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=121&option=3</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>121</td>
      <td>Video Digital Lenses</td>
      <td>Set video digital lenses (id: 121) to linear (id: 4)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=121&option=4</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>121</td>
      <td>Video Digital Lenses</td>
      <td>Set video digital lenses (id: 121) to max superview (id: 7)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=121&option=7</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>121</td>
      <td>Video Digital Lenses</td>
      <td>Set video digital lenses (id: 121) to linear + horizon leveling (id: 8)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=121&option=8</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>122</td>
      <td>Photo Digital Lenses</td>
      <td>Set photo digital lenses (id: 122) to narrow (id: 19)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=122&option=19</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>122</td>
      <td>Photo Digital Lenses</td>
      <td>Set photo digital lenses (id: 122) to max superview (id: 100)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=122&option=100</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>122</td>
      <td>Photo Digital Lenses</td>
      <td>Set photo digital lenses (id: 122) to wide (id: 101)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=122&option=101</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>122</td>
      <td>Photo Digital Lenses</td>
      <td>Set photo digital lenses (id: 122) to linear (id: 102)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=122&option=102</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>123</td>
      <td>Time Lapse Digital Lenses</td>
      <td>Set time lapse digital lenses (id: 123) to narrow (id: 19)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=123&option=19</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>123</td>
      <td>Time Lapse Digital Lenses</td>
      <td>Set time lapse digital lenses (id: 123) to max superview (id: 100)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=123&option=100</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>123</td>
      <td>Time Lapse Digital Lenses</td>
      <td>Set time lapse digital lenses (id: 123) to wide (id: 101)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=123&option=101</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(245,249,255);">
      <td>123</td>
      <td>Time Lapse Digital Lenses</td>
      <td>Set time lapse digital lenses (id: 123) to linear (id: 102)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=123&option=102</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>173</td>
      <td>Video Performance Mode</td>
      <td>Set video performance mode (id: 173) to maximum video performance (id: 0)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=173&option=0</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>173</td>
      <td>Video Performance Mode</td>
      <td>Set video performance mode (id: 173) to extended battery (id: 1)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=173&option=1</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
    <tr style="background-color: rgb(222,235,255);">
      <td>173</td>
      <td>Video Performance Mode</td>
      <td>Set video performance mode (id: 173) to tripod / stationary video (id: 2)</td>
      <td>GET</td>
      <td>/gopro/camera/setting?setting=173&option=2</td>
      <td>\&gt;= v01.16.00</td>
    </tr>
  </tbody>
</table>


### Camera Capabilities
Below are tables detailing supported features for Open GoPro cameras.

#### HERO10 Black
<table border="1">
  <tbody>
    <tr style="background-color: rgb(0,0,0); color: rgb(255,255,255);">
      <td>Resolution</td>
      <td>Anti-Flicker</td>
      <td>Frames Per Second</td>
      <td>Video Digital Lenses</td>
    </tr>
    <tr>
      <td rowspan="38">1080</td>
      <td rowspan="19">50Hz</td>
      <td rowspan="5">25</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">50</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">100</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">200</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="19">60Hz</td>
      <td rowspan="5">30</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">60</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">120</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">240</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="28">2.7K</td>
      <td rowspan="14">50Hz</td>
      <td rowspan="5">50</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">100</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">200</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="14">60Hz</td>
      <td rowspan="5">60</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">120</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">240</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="16">2.7K 4:3</td>
      <td rowspan="8">50Hz</td>
      <td rowspan="4">50</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">100</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="8">60Hz</td>
      <td rowspan="4">60</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">120</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="38">4K</td>
      <td rowspan="19">50Hz</td>
      <td rowspan="5">24</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">25</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">50</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">100</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="19">60Hz</td>
      <td rowspan="5">24</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">30</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="5">60</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Superview</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">120</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="8">4K 4:3</td>
      <td rowspan="4">50Hz</td>
      <td rowspan="4">50</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">60Hz</td>
      <td rowspan="4">60</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="16">5.3K</td>
      <td rowspan="8">50Hz</td>
      <td rowspan="4">25</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">50</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="8">60Hz</td>
      <td rowspan="4">30</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">60</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="8">5K 4:3</td>
      <td rowspan="4">50Hz</td>
      <td rowspan="4">25</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
    <tr>
      <td rowspan="4">60Hz</td>
      <td rowspan="4">30</td>
      <td>Wide</td>
    </tr>
    <tr>
      <td>Narrow</td>
    </tr>
    <tr>
      <td>Linear</td>
    </tr>
    <tr>
      <td>Linear + Horizon Leveling</td>
    </tr>
  </tbody>
</table>


# Media
<p>
The camera provides an endpoint to query basic details about media captured on the sdcard.
</p>


## Chapters
<p>
All GoPro cameras break longer videos into chapters.
GoPro cameras currently limit file sizes on sdcards to 4GB for both FAT32 and exFAT file systems.
This limitation is most commonly seen when recording longer (10+ minute) videos.
In practice, the camera will split video media into chapters named Gqccmmmm.MP4 (and ones for THM/LRV) such that:
</p>

<ul>
<li>q: Quality Level (X: Extreme, H: High, M: Medium, L: Low)</li>
<li>cc: Chapter Number (01-99)</li>
<li>mmmm: Media ID (0001-9999)</li>
</ul>

<p>
When media becomes chaptered, the camera increments subsequent Chapter Numbers while leaving the Media ID unchanged.
For example, if the user records a long High-quality video that results in 4 chapters, the files on the sdcard may
look like the following:
</p>

```
-rwxrwxrwx@ 1 gopro  123456789  4006413091 Jan  1 00:00 GH010078.MP4
-rwxrwxrwx@ 1 gopro  123456789       17663 Jan  1 00:00 GH010078.THM
-rwxrwxrwx@ 1 gopro  123456789  4006001541 Jan  1 00:00 GH020078.MP4
-rwxrwxrwx@ 1 gopro  123456789       17357 Jan  1 00:00 GH020078.THM
-rwxrwxrwx@ 1 gopro  123456789  4006041985 Jan  1 00:00 GH030078.MP4
-rwxrwxrwx@ 1 gopro  123456789       17204 Jan  1 00:00 GH030078.THM
-rwxrwxrwx@ 1 gopro  123456789   756706872 Jan  1 00:00 GH040078.MP4
-rwxrwxrwx@ 1 gopro  123456789       17420 Jan  1 00:00 GH040078.THM
-rwxrwxrwx@ 1 gopro  123456789   184526939 Jan  1 00:00 GL010078.LRV
-rwxrwxrwx@ 1 gopro  123456789   184519787 Jan  1 00:00 GL020078.LRV
-rwxrwxrwx@ 1 gopro  123456789   184517614 Jan  1 00:00 GL030078.LRV
-rwxrwxrwx@ 1 gopro  123456789    34877660 Jan  1 00:00 GL040078.LRV
```


## Media List Format
<p>
The format of the media list is given below.
</p>

```
{
    "id": "<MEDIA SESSION ID>",
    "media": [
        {
            "d": "<DIRECTORY NAME>",
            "fs": [
                {<MEDIA ITEM INFO>},
                ...
            ]
        },
        ...
    ]
}
```

### Media List Keys
The outer structure of the media list and the inner structure of individual media items use the keys in the table below.
<table border="1">
  <tbody>
    <tr style="background-color: rgb(0,0,0); color: rgb(255,255,255);">
      <td>Key</td>
      <td>Description</td>
    </tr>
    <tr>
      <td>b</td>
      <td>ID of first member of a group (for grouped media items)</td>
    </tr>
    <tr>
      <td>d</td>
      <td>Directory name</td>
    </tr>
    <tr>
      <td>fs</td>
      <td>File system. Contains listing of media items in directory</td>
    </tr>
    <tr>
      <td>g</td>
      <td>Group ID (if grouped media item)</td>
    </tr>
    <tr>
      <td>id</td>
      <td>Media list session identifier</td>
    </tr>
    <tr>
      <td>l</td>
      <td>ID of last member of a group (for grouped media items)</td>
    </tr>
    <tr>
      <td>ls</td>
      <td>Low resolution video file size</td>
    </tr>
    <tr>
      <td>m</td>
      <td>List of missing/deleted group member IDs (for grouped media items)</td>
    </tr>
    <tr>
      <td>media</td>
      <td>Contains media info for for each directory (e.g. 100GOPRO/, 101GOPRO/, ...)</td>
    </tr>
    <tr>
      <td>mod</td>
      <td>Last modified time (seconds since epoch)</td>
    </tr>
    <tr>
      <td>n</td>
      <td>Media filename</td>
    </tr>
    <tr>
      <td>s</td>
      <td>Size of (group) media in bytes</td>
    </tr>
    <tr>
      <td>t</td>
      <td>Group type (for grouped media items) (b -> burst, c -> continuous shot, n -> night lapse, t -> time lapse)</td>
    </tr>
  </tbody>
</table>

### Grouped Media Items
<p>
In order to minimize the size of the JSON transmitted by the camera, grouped media items such as Burst Photos,
Time Lapse Photos, Night Lapse Photos, etc are represented with a single item in the media list with additional keys
that allow the user to extrapolate individual filenames for each member of the group.
</p>

<p>
Filenames for group media items have the form "GXXXYYYY.ZZZ"
where XXX is the group ID, YYY is the group member ID and ZZZ is the file extension.
</p>

<p>
For example, take the media list below, which contains a Time Lapse Photo group media item:
</p>

```
{
    "id": "2530266050123724003",
    "media": [
        {
            "d": "100GOPRO",
            "fs": [
                {
                    "b": "8",
                    "cre": "1613669353",
                    "g": "1",
                    "l": "396",
                    "m": ['75', '139'],
                    "mod": "1613669353",
                    "n": "G0010008.JPG",
                    "s": "773977407",
                    "t": "t"
                }
            ]
        }
    ]
}
```

<p>
The first filename in the group is "G0010008.JPG" (key: "n").<br/>
The ID of the first group member in this case is "008" (key: "b").<br/>
The ID of the last group member in this case is "396" (key: "l").<br/>
The IDs of deleted members in this case are "75" and "139" (key: "m")<br/>
Given this information, the user can extrapolate that the group currently contains
</p>

<p>
G0010008.JPG, G0010009.JPG, G0010010.JPG,<br/>
...,<br/>
G0010074.JPG, G0010076.JPG,<br/>
...,<br/>
G0010138.JPG, G0010140.JPG,<br/>
...,<br/>
G0010394.JPG, G0010395.JPG. G0010396.JPG<br/>
</p>



## Downloading Media
<p>
The URL to download/stream media from the DCIM/ directory on the sdcard is the Base URL plus <i>/videos/DCIM/XXX/YYY</i>
where XXX is the directory name within DCIM/ given by the media list and YYY is the target media filename.
</p>

<p>
For example: Given the following media list:
</p>

```
{
    "id": "3586667939918700960",
    "media": [
        {
            "d": "100GOPRO",
            "fs": [
                {
                    "n": "GH010397.MP4",
                    "cre": "1613672729",
                    "mod": "1613672729",
                    "glrv": "1895626",
                    "ls": "-1",
                    "s": "19917136"
                },
                {
                    "cre": "1614340213",
                    "mod": "1614340213",
                    "n": "GOPR0001.JPG",
                    "s": "6961371"
                }
            ]
        }
    ]
}
```

<p>
The URL to download GH010397.MP4 over WiFi would be
<a href="http://10.5.5.9:8080/videos/DCIM/100GOPRO/GH010397.MP4">http://10.5.5.9:8080/videos/DCIM/100GOPRO/GH010397.MP4</a>
</p>

<p>
The URL to download GOPR0001.JPG over WiFi would be
<a href="http://10.5.5.9:8080/videos/DCIM/100GOPRO/GOPR0001.JPG">http://10.5.5.9:8080/videos/DCIM/100GOPRO/GOPR0001.JPG</a>
</p>


## Turbo Transfer
<p>
Some cameras support Turbo Transfer mode, which allows media to be downloaded over WiFi more rapidly.
This special mode should only be used during media offload.
It is recommended that the user check for and--if necessary--disable Turbo Transfer on connect.
For details on which cameras are supported and how to enable and disable Turbo Transfer, see
<a href="#commands-quick-reference">Commands Quick Reference</a>.
</p>


## Downloading Preview Stream
<p>
When the preview stream is started, the camera starts up a UDP client and begins writing MPEG Transport Stream
data to the connected client on port 8554.
In order to stream and save this data, the user can implement a UDP server that binds to the same port and appends
datagrams to a file when they are received.
</p>


# Camera State
<p>
The camera provides multiple types of state, all of which can be queried:
</p>

<ul>
<li>Camera state: Contains information about camera status (photos taken, date, is-camera-encoding, etc) and settings (current video resolution, current frame rate, etc)</li>
<li>Preset State: How presets are arranged into preset groups, their titles, icons, settings closely associated with each preset, etc</li>
</ul>


## Camera State Format
Camera state is given in the following form:

```
{
    "status": {
        "1": <status 1 value>,
        "2": <status 2 value>,
        ...
    },
    "settings: {
        "2": <setting 2 value>,
        "3": <setting 3 value>,
        ...
    }
}
```

<p>
Where <b>status X value</b> and <b>setting X value</b> are almost always integer values.
See Status Codes table in this document for exceptions.
</p>

<p>
For status, keys are status codes and values are status values.
</p>

<p>
For settings, keys are setting IDs, and values are option values
</p>


## Status Codes
<p>
Below is a table of supported status codes.<br />
<span style="color:green">✔</span> Indicates support for all Open GoPro firmware versions.<br />
<span style="color:red">❌</span> Indicates a lack of support for all Open GoPro firmware versions.<br />
>= vXX.YY.ZZ indicates support for firmware versions equal to or newer than vXX.YY.ZZ
</p>

<table border="1">
  <tbody>
    <tr style="background-color: rgb(0,0,0); color: rgb(255,255,255);">
      <td>Status ID</td>
      <td>Name</td>
      <td>Description</td>
      <td>Type</td>
      <td>Values</td>
      <td>HERO10 Black</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Internal battery present</td>
      <td>Is the system's internal battery present?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>2</td>
      <td>Internal battery level</td>
      <td>Rough approximation of internal battery level in bars</td>
      <td>integer</td>
      <td>0: Zero<br />1: One<br />2: Two<br />3: Three<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>3</td>
      <td>External battery present</td>
      <td>Is an external battery connected?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>4</td>
      <td>External battery level</td>
      <td>External battery power level in percent</td>
      <td>percent</td>
      <td>0-100</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>6</td>
      <td>System hot</td>
      <td>Is the system currently overheating?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>8</td>
      <td>System busy</td>
      <td>Is the camera busy?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>9</td>
      <td>Quick capture active</td>
      <td>Is Quick Capture feature enabled?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>10</td>
      <td>Encoding active</td>
      <td>Is the system encoding right now?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>11</td>
      <td>Lcd lock active</td>
      <td>Is LCD lock active?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>13</td>
      <td>Video progress counter</td>
      <td>When encoding video, this is the duration (seconds) of the video so far; 0 otherwise</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>17</td>
      <td>Enable</td>
      <td>Are Wireless Connections enabled?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>19</td>
      <td>State</td>
      <td>The pairing state of the camera</td>
      <td>integer</td>
      <td>0: Success<br />1: In Progress<br />2: Failed<br />3: Stopped<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>20</td>
      <td>Type</td>
      <td>The last type of pairing that the camera was engaged in</td>
      <td>integer</td>
      <td>0: Not Pairing<br />1: Pairing App<br />2: Pairing Remote Control<br />3: Pairing Bluetooth Device<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>21</td>
      <td>Pair time</td>
      <td>Time (milliseconds) since boot of last successful pairing complete action</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>22</td>
      <td>State</td>
      <td>State of current scan for WiFi Access Points. Appears to only change for CAH-related scans</td>
      <td>integer</td>
      <td>0: Never started<br />1: Started<br />2: Aborted<br />3: Canceled<br />4: Completed<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>23</td>
      <td>Scan time msec</td>
      <td>The time, in milliseconds since boot that the WiFi Access Point scan completed</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>24</td>
      <td>Provision status</td>
      <td>WiFi AP provisioning state</td>
      <td>integer</td>
      <td>0: Never started<br />1: Started<br />2: Aborted<br />3: Canceled<br />4: Completed<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>26</td>
      <td>Remote control version</td>
      <td>Wireless remote control version</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>27</td>
      <td>Remote control connected</td>
      <td>Is a wireless remote control connected?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>28</td>
      <td>Pairing</td>
      <td>Wireless Pairing State</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>29</td>
      <td>Wlan ssid</td>
      <td>Provisioned WIFI AP SSID. On BLE connection, value is big-endian byte-encoded int</td>
      <td>string</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>30</td>
      <td>Ap ssid</td>
      <td>Camera's WIFI SSID. On BLE connection, value is big-endian byte-encoded int</td>
      <td>string</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>31</td>
      <td>App count</td>
      <td>The number of wireless devices connected to the camera</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>32</td>
      <td>Enable</td>
      <td>Is Preview Stream enabled?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>33</td>
      <td>Sd status</td>
      <td>Primary Storage Status</td>
      <td>integer</td>
      <td>-1: Unknown<br />0: OK<br />1: SD Card Full<br />2: SD Card Removed<br />3: SD Card Format Error<br />4: SD Card Busy<br />8: SD Card Swapped<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>34</td>
      <td>Remaining photos</td>
      <td>How many photos can be taken before sdcard is full</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>35</td>
      <td>Remaining video time</td>
      <td>How many minutes of video can be captured with current settings before sdcard is full</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>36</td>
      <td>Num group photos</td>
      <td>How many group photos can be taken with current settings before sdcard is full</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>37</td>
      <td>Num group videos</td>
      <td>Total number of group videos on sdcard</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>38</td>
      <td>Num total photos</td>
      <td>Total number of photos on sdcard</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>39</td>
      <td>Num total videos</td>
      <td>Total number of videos on sdcard</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>40</td>
      <td>Date time</td>
      <td>Current date/time (format: %YY%MM%DD%HH%MM%SS, all values in hex)</td>
      <td>string</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>41</td>
      <td>Ota status</td>
      <td>The current status of Over The Air (OTA) update</td>
      <td>integer</td>
      <td>0: Idle<br />1: Downloading<br />2: Verifying<br />3: Download Failed<br />4: Verify Failed<br />5: Ready<br />6: GoPro App: Downloading<br />7: GoPro App: Verifying<br />8: GoPro App: Download Failed<br />9: GoPro App: Verify Failed<br />10: GoPro App: Ready<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>42</td>
      <td>Download cancel request pending</td>
      <td>Is there a pending request to cancel a firmware update download?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>45</td>
      <td>Camera locate active</td>
      <td>Is locate camera feature active?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>49</td>
      <td>Multi shot count down</td>
      <td>The current timelapse interval countdown value (e.g. 5...4...3...2...1...)</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>54</td>
      <td>Remaining space</td>
      <td>Remaining space on the sdcard in Kilobytes</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>55</td>
      <td>Supported</td>
      <td>Is preview stream supported in current recording/flatmode/secondary-stream?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>56</td>
      <td>Wifi bars</td>
      <td>WiFi signal strength in bars</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>57</td>
      <td>Current time msec</td>
      <td>System time in milliseconds since system was booted</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>58</td>
      <td>Num hilights</td>
      <td>The number of hilights in encoding video (set to 0 when encoding stops)</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>59</td>
      <td>Last hilight time msec</td>
      <td>Time since boot (msec) of most recent hilight in encoding video (set to 0 when encoding stops)</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>60</td>
      <td>Next poll msec</td>
      <td>The min time between camera status updates (msec). Do not poll for status more often than this</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>64</td>
      <td>Remaining timelapse time</td>
      <td>How many min of Timelapse video can be captured with current settings before sdcard is full</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>65</td>
      <td>Exposure select type</td>
      <td>Liveview Exposure Select Mode</td>
      <td>integer</td>
      <td>0: Disabled<br />1: Auto<br />2: ISO Lock<br />3: Hemisphere<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>66</td>
      <td>Exposure select x</td>
      <td>Liveview Exposure Select: y-coordinate (percent)</td>
      <td>percent</td>
      <td>0-100</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>67</td>
      <td>Exposure select y</td>
      <td>Liveview Exposure Select: y-coordinate (percent)</td>
      <td>percent</td>
      <td>0-100</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>68</td>
      <td>Gps status</td>
      <td>Does the camera currently have a GPS lock?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>69</td>
      <td>Ap state</td>
      <td>Is the WiFi radio enabled?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>70</td>
      <td>Internal battery percentage</td>
      <td>Internal battery level (percent)</td>
      <td>percent</td>
      <td>0-100</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>74</td>
      <td>Acc mic status</td>
      <td>Microphone Accesstory status</td>
      <td>integer</td>
      <td>0: Microphone mod not connected<br />1: Microphone mod connected<br />2: Microphone mod connected and microphone plugged into Microphone mod<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>75</td>
      <td>Digital zoom</td>
      <td>Digital Zoom level (percent)</td>
      <td>percent</td>
      <td>0-100</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>76</td>
      <td>Wireless band</td>
      <td>Wireless Band</td>
      <td>integer</td>
      <td>0: 2.4 GHz<br />1: 5 GHz<br />2: Max<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>77</td>
      <td>Digital zoom active</td>
      <td>Is Digital Zoom feature available?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>78</td>
      <td>Mobile friendly video</td>
      <td>Are current video settings mobile friendly? (related to video compression and frame rate)</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>79</td>
      <td>First time use</td>
      <td>Is the camera currently in First Time Use (FTU) UI flow?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>81</td>
      <td>Band 5ghz avail</td>
      <td>Is 5GHz wireless band available?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>82</td>
      <td>System ready</td>
      <td>Is the system ready to accept commands?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>83</td>
      <td>Batt okay for ota</td>
      <td>Is the internal battery charged sufficiently to start Over The Air (OTA) update?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>85</td>
      <td>Video low temp alert</td>
      <td>Is the camera getting too cold to continue recording?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>86</td>
      <td>Actual orientation</td>
      <td>The rotational orientation of the camera</td>
      <td>integer</td>
      <td>0: 0 degrees (upright)<br />1: 180 degrees (upside down)<br />2: 90 degrees (laying on right side)<br />3: 270 degrees (laying on left side)<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>88</td>
      <td>Zoom while encoding</td>
      <td>Is this camera capable of zooming while encoding (static value based on model, not settings)</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>89</td>
      <td>Current mode</td>
      <td>Current flatmode ID</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>91</td>
      <td>Logs ready</td>
      <td>Are system logs ready to be downloaded?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>93</td>
      <td>Active video presets</td>
      <td>Current Video Preset (ID)</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>94</td>
      <td>Active photo presets</td>
      <td>Current Photo Preset (ID)</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>95</td>
      <td>Active timelapse presets</td>
      <td>Current Timelapse Preset (ID)</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>96</td>
      <td>Active presets group</td>
      <td>Current Preset Group (ID)</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>97</td>
      <td>Active preset</td>
      <td>Current Preset (ID)</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>98</td>
      <td>Preset modified</td>
      <td>Preset Modified Status, which contains an event ID and a preset (group) ID</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>99</td>
      <td>Remaining live bursts</td>
      <td>How many Live Bursts can be captured before sdcard is full</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>100</td>
      <td>Num total live bursts</td>
      <td>Total number of Live Bursts on sdcard</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>101</td>
      <td>Capture delay active</td>
      <td>Is Capture Delay currently active (i.e. counting down)?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>102</td>
      <td>Media mod mic status</td>
      <td>Media mod State</td>
      <td>integer</td>
      <td>0: Media mod microphone removed<br />2: Media mod microphone only<br />3: Media mod microphone with external microphone<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>103</td>
      <td>Timewarp speed ramp active</td>
      <td>Time Warp Speed</td>
      <td>integer</td>
      <td>0: 15x<br />1: 30x<br />2: 60x<br />3: 150x<br />4: 300x<br />5: 900x<br />6: 1800x<br />7: 2x<br />8: 5x<br />9: 10x<br />10: Auto<br />11: 1x (realtime)<br />12: 1/2x (slow-motion)<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>104</td>
      <td>Linux core active</td>
      <td>Is the system's Linux core active?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>105</td>
      <td>Camera lens type</td>
      <td>Camera lens type (reflects changes to setting 162)</td>
      <td>integer</td>
      <td>0: Default<br />1: Max Lens<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>106</td>
      <td>Video hindsight capture active</td>
      <td>Is Video Hindsight Capture Active?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>107</td>
      <td>Scheduled preset</td>
      <td>Scheduled Capture Preset ID</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>108</td>
      <td>Scheduled enabled</td>
      <td>Is Scheduled Capture set?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>109</td>
      <td>Creating preset</td>
      <td>Is the camera in the process of creating a custom preset?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>110</td>
      <td>Media mod status</td>
      <td>Media Mode Status (bitmasked)</td>
      <td>integer</td>
      <td>0: Display (selfie) mod: 0, HDMI: 0, Media Mod Connected: False<br />1: Display (selfie) mod: 0, HDMI: 0, Media Mod Connected: True<br />2: Display (selfie) mod: 0, HDMI: 1, Media Mod Connected: False<br />3: Display (selfie) mod: 0, HDMI: 1, Media Mod Connected: True<br />4: Display (selfie) mod: 1, HDMI: 0, Media Mod Connected: False<br />5: Display (selfie) mod: 1, HDMI: 0, Media Mod Connected: True<br />6: Display (selfie) mod: 1, HDMI: 1, Media Mod Connected: False<br />7: Display (selfie) mod: 1, HDMI: 1, Media Mod Connected: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>111</td>
      <td>Sd rating check error</td>
      <td>Does sdcard meet specified minimum write speed?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>112</td>
      <td>Sd write speed error</td>
      <td>Number of sdcard write speed errors since device booted</td>
      <td>integer</td>
      <td>*</td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>113</td>
      <td>Turbo transfer</td>
      <td>Is Turbo Transfer active?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
    <tr>
      <td>115</td>
      <td>Usb connected</td>
      <td>Is the camera connected to a PC via USB?</td>
      <td>boolean</td>
      <td>0: False<br />1: True<br /></td>
      <td><span style="color:green">✔</span></td>
    </tr>
  </tbody>
</table>


## Preset Status Format
<p>
Preset Status is returned as JSON, whose content is the serialization of the <a href="https://developers.google.com/protocol-buffers">protobuf</a> message:
<a href="https://github.com/gopro/OpenGoPro/blob/main/protobuf/preset_status.proto">NotifyPresetStatus</a>.
Using Google protobuf APIs, the JSON can be converted back into a programmatic object in the user's language of choice.
</p>
