I have baked a custom integration to control Eufy Security Cameras and access RSTP (real time streaming protocol) stream if possible. You can turn on and turn off cameras and if your camera is on, you can view live stream. Morevoer, there are some additional sensors for motion detection, person detection, battery level and wifi signal. 

**Big thanks to @bropat who made this possible. Please consider buying a coffee for him over here: https://ko-fi.com/bropat**

# 1. Services #
## 1.1 Camera Services ##
- start_rtsp / stop_rtsp (RTSP): **if your camera can stream over RTSP, please enable it over eufy app, this is more more reliable and less power consuming and you can use these services to start and stop stream.**. Attention: users had reported that using `stop_rtsp` is disabling RTSP functionality at all for the camera. https://github.com/fuatakgun/eufy_security/issues/53 
- start_livesteam / stop_livestream (P2P): if there is no support for RTSP, you can use P2P streaming, this should work for all camera types but much more power consuming for your HA instance.
- turn_on / turn_off: it first checks **if there is rtsp attribute in camera and if yes; it will use RTSP services,** if not, it will use P2P services.
- enable / disable: enable and disable respective device

## 1.2 Station Services ##
![image](https://user-images.githubusercontent.com/11085566/127906780-ba18d5a0-03c3-407a-922a-dc519e59dfe8.png)
- alarm_arm_home / alarm_arm_away / alarm_disarm / alarm_guard_schedule (requires configuration on eufy app) / alarm_guard_geo (requires configuration on eufy app)
- alarm_arm_custom1 / alarm_arm_custom2 / alarm_arm_custom3 - you need to create custom configurations for these to work
- alarm_trigger / alarm_trigger_with_duration - trigger alarm on home station
- reset_alarm - stop the alarm state

## 1.2 Integration Services ##
- force_sync - get latest changes from cloud as some changes are not generating notifications to be captured automatically

# 2. Known Bugs / Issues #
Please throw some :)

# 3. Troubleshooting
1- Create a separate account for HA integration as that account will be logged out automatically from mobile app when HA integration logged in. Do not forget to share your cameras with your new account and enable notifications for them. This integration depends on push notifications to catch events.

2- RTSP - As of now, live stream is limited to 3 minutes and this is a hard limitation by Eufy, so we do not have a solution in place. So, if you keep live stream running more than 3 minutes, it will be turned off by hardware but your home assistant will not be notified on this. So, next time you want to start live stream, you notice that nothing will be happening as we assume that it is already running. As a workaround, please call stop and start in order. https://github.com/fuatakgun/eufy_security/issues/10#issuecomment-886251442 

3- P2P - To have P2P streaming work out, we have an additional add-on to mirror incoming video bytes and stream as it is an RTSP stream. But to do so, integration first needs to analyze X seconds from incmoing bytes to understand video codec information (dimensions, fps, codec etc) and then initializes the stream on add-on. So, depending on your hardware and video quality this could change between 1 to 5 seconds. If your P2P stream fails to start, please play with this configuration in integration options page. Check below image;

![image](https://user-images.githubusercontent.com/11085566/136794616-fa238dd8-9bd6-41d8-ac14-2c0fb4f0eb23.png)

4- Please do not enable `Preload Stream` functionality in Camera View if your camera is battery powered (not streaming all the time) for two reasons; it is not adding functionality in our use case and it sends a signal to enable live stream to your cameras which might end up excessive battery consumption.
![image](https://user-images.githubusercontent.com/11085566/128697823-f83b5ce5-1f31-48c9-ac6d-712bd56b504b.png)

5- One user reported that there is an issue regarding to 2k Battery Doorbell in terms of receiving motion and person detection sensor. If you are having a similar issue, please apply this solution: https://github.com/fuatakgun/eufy_security/issues/22#issuecomment-908157691

6- I am more than happy to debug individual issues as long as you follow setup instructions. I need you to share your problematic cameras with me so that I can use my own machine to debug the issue. For each debugging request, please create a github issue so we can track from there. Do not forget to remove the sharing settings after we are done :)

- If you are located in EU, use my account: `fuatakgun@gmail.com`
- If you are located in US, use shared test account: `eufydeveloper@gmail.com`

6- If you have any other issue, please create it an issue on github repository, give information about your home assistant hardware, camera model, streaming type (rtsp or p2p), steps required to generate the issue. Enable excessive logging and share your logs from integration and related add ons.

```
logger:
  default: info
  logs:
    custom_components.eufy_security: debug
```

# 6. Installation
***Warning, there is an existing integration (https://github.com/nonsleepr/ha-eufy-security) and I have used the same internal name with this integration, unintentinally. You can not keep both integrations and trying to install might cause issues. You can backup old one if you want to test this, just rename `custom_components/eufy_security` into something else (eg `eufy_security_nonsleepr`)***

Please follow screenshots below. In summary;
- You will first install HASS Add On assuming you are running on Hassos or Supervised. If not, please execute this command to run docker instance manually ```docker run -it -e USERNAME=email@address.com -e PASSWORD=password_goes_here -p 3000:3000 bropat/eufy-security-ws:latest```
- Later on, you should install RTSP Server Add On to have faster/more reliable p2p streaming. I will deprecate/not support file based streaming soon, so, please migrate in timely manner. If you are not using Hassos or Supervised installation please execute this command to run docker instance manually ```docker run --rm -it -e RTSP_PROTOCOLS=tcp -d -p 8554:8554 -p 1935:1935 aler9/rtsp-simple-server```
- When you are done with HASS Add On, you will install integration via adding integration address to HACS, downloading files over UI, restarting home assistant and setting up integration.
- Double check if your `configuration.yaml` includes `ffmpeg` integration. If not, please do like this; https://www.home-assistant.io/integrations/ffmpeg/#configuration . This integration relies on `ffmpeg` to be setup.

## 6.1 Installing Eufy Security Add On
1- Go to Add-On Store page and select `Repositories`

![1-add-on-store](https://user-images.githubusercontent.com/11085566/126563889-8bc98e9a-8cb5-4f71-a3a7-3bde8e3f1182.PNG)

2- Add custom repository URL 
```https://github.com/fuatakgun/eufy_security_addon```

![2-add-on-repository](https://user-images.githubusercontent.com/11085566/126563898-8c642026-1e16-4484-8177-0bc6a93d59e8.PNG)

3- Confirm that you can see `Eufy Security WS Addon`

![3-add-on-visible](https://user-images.githubusercontent.com/11085566/126563911-ec5e0e52-312b-4e65-a25b-54a02a348752.PNG)

4- Click on `Eufy Security WS Addon`, install add-on and switch to `Configuration` page, fill username, password and **country code (2 letter)**. I advise you to create a new account and share your cameras from main account to new account. Use that new account for home assistant purposes only.

![4-add-on-configure](https://user-images.githubusercontent.com/11085566/126563919-273e413b-f2ac-49c4-8342-dfd5c5887ccf.PNG)

5-Validate that you are connected checking logs page.

![5-add-on-log](https://user-images.githubusercontent.com/11085566/126563928-3ee2d48d-06e2-4681-9076-3992f4546b16.PNG)

## 6.2 Installing RTSP Simple Server Add On (for faster P2P streaming)
1- Go to Add-On Store page and select `Repositories`
![1-add-on-store](https://user-images.githubusercontent.com/11085566/126563889-8bc98e9a-8cb5-4f71-a3a7-3bde8e3f1182.PNG)

2- Add custom repository URL 
```https://github.com/fuatakgun/rtsp_simple_server```

3- Confirm that you can see `RTSP Simple Server Addon` - probably at the end of the page
![image](https://user-images.githubusercontent.com/11085566/127865866-5c47cfd1-0130-4a6a-a00c-8a763acd2100.png)

4- Click on `RTSP Simple Server Addon`, install add-on, please do not change any configuration.
![image](https://user-images.githubusercontent.com/11085566/127866038-44d2db72-2e20-46bd-a3d7-328213bf6713.png)

5- Start the Add-On and validate if it is running well checking the logs.
![image](https://user-images.githubusercontent.com/11085566/127866173-af817b84-034e-449e-8143-a94a78564052.png)

## 6.3 Installing Integration

1- Go to HACS and click on `Custom repositories`

![6-hacs-custom-repositories](https://user-images.githubusercontent.com/11085566/126563932-e9fc2783-02a1-42d3-8f4a-bc0fa2edf386.PNG)

2- Add custom integration repository URL 
```https://github.com/fuatakgun/eufy_security/```

![7-hacs-add](https://user-images.githubusercontent.com/11085566/126563937-4ad08d92-b9c1-45e3-a205-be9b244bc3a7.PNG)

3- Install the `Eufy Secucirty` repository

![8-hacs-install](https://user-images.githubusercontent.com/11085566/126563950-1c89c1e8-f77d-46ac-8910-77048500a07f.PNG)

4- You need to restart your HA instance to pick up new repository.

![9-hacs-restart](https://user-images.githubusercontent.com/11085566/126563954-b801e4ea-b93e-4695-928d-a82221fe01f4.PNG)

5- After restart, go to `Configuration` - `Integrations` page and click on `Add Integration`

![10-integrations](https://user-images.githubusercontent.com/11085566/126563961-a05c5e50-b006-4759-b55a-548f691a13d8.PNG)

6- Search for ```Eufy Security``` and click on it

![11-integration-search](https://user-images.githubusercontent.com/11085566/126563968-920a74de-ab93-456b-b4b2-dcf651a07f9f.PNG)

7- In next page, use ```localhost``` if you have used Add-on installation, otherwise put your Docker instance ip address and keep `3000` as port

![12-integration-configure](https://user-images.githubusercontent.com/11085566/126563976-234005e7-2920-4ef0-a301-187d4d929f10.png)

8- You will be shown devices connected to your account.

![13-integration-done](https://user-images.githubusercontent.com/11085566/126563982-38b3a00a-ff6a-45aa-8dcc-b04e864a37f8.PNG)

9- If your camera does not support RTSP based live streaming, you can use `Start Live Stream` and `Stop Live Stream` services rather than turn_on and turn_off because they tend to be using RTSP functions. They require camera entities as input, you can use UI for this.

![14-services-live-stream](https://user-images.githubusercontent.com/11085566/126563991-5ef949c5-144c-4702-a9e3-577e2d37c0f8.PNG)

10- If you want faster P2P live streaming, go to Integration Configuration section and enable it.
![image](https://user-images.githubusercontent.com/11085566/127866543-1345d56f-b4f3-4154-96c7-a278d747cf8d.png)

## 6.3 Optional - WebRTC

You can use WebRTC for light speed streaming inside Home Assistant.

1- Install WebRTC following these steps: https://github.com/AlexxIT/WebRTC#installation

2- Disable Auto Start on Click to be sure that cameras are not starting to stream autoamtically.

![image](https://user-images.githubusercontent.com/11085566/136706616-7ba09fec-f75b-4010-b58e-bf0f8f2d47da.png)

3- Setup two conditional cards for each camera as below, do not forget to put correct camera entity names (replace `entrance` with your camera name)
- So, when camera is not streaming, you will get latest captured image and when you click on image, it will start streaming (camera.turn_on service call).
- When camera is streaming, you will get WebRTC card which has 1-2 seconds latency while streaming.

```
- type: conditional
  conditions:
    - entity: binary_sensor.entrance_streaming_sensor
      state: 'False'
  card:
    type: picture-entity
    entity: camera.entrance_camera
    tap_action:
      action: call-service
      service: camera.turn_on
      service_data: {}
      target:
        entity_id: camera.entrance_camera
- type: conditional
  conditions:
    - entity: binary_sensor.entrance_streaming_sensor
      state: 'True'
  card:
    type: custom:webrtc-camera
    entity: camera.entrance_camera
```

RTSP Experience with WebRTC: https://drive.google.com/file/d/1qIYUx82C0CnpsTycP9dTS0NX6IEqeqHD/view?usp=drivesdk

P2P Experience with WebRTC: https://drive.google.com/file/d/1qCW9XX32vInQgFF7hPqoxp-ssno2Sye9/view?usp=drivesdk

Thanks @conorlap for this. - https://github.com/fuatakgun/eufy_security/issues/43



### Raise your issues in Github. ###
