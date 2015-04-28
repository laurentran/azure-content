<properties 
	pageTitle="Working with Channels that are Enabled to Perform Live Encoding with Azure Media Services" 
	description="This topic describes how to set up a Channel that receives a single bitrate live stream from an on-premises encoder and then performs live encoding to adaptive bitrate stream with Media Services. The stream can then be delivered to client playback applications through one or more Streaming Endpoints, using one of the following adaptive streaming protocols: HLS, Smooth Stream, MPEG DASH, HDS." 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="ne" 
	ms.topic="article" 
	ms.date="04/06/2015" 
	ms.author="juliako"/>

#Working with Channels that are Enabled to Perform Live Encoding with Azure Media Services(Preview)

##Overview

In Azure Media Services (AMS), a **Channel** represents a pipeline for processing live streaming content. A Channel receives live input streams in one of two ways:

- An on-premises live encoder sends multi-bitrate RTMP or Smooth Streaming (Fragmented MP4) to the Channel. The Channel does not process the stream. When requested, Media Services delivers the stream to customers. 
- A single bitrate RTMP, Smooth Streaming (Fragmented MP4), or RTP (MPEG-TS) stream is sent to a channel that is enabled to perform live encoding with Media Services. The Channel then encodes the incoming stream to a multi-bitrate video stream. When requested, Media Services delivers the stream to customers. 

	Encoding a live stream with Media Services is currently in Preview.

Starting with the Media Services 2.10 release, when you create a Channel, you can specify in which way you want for your channel to receive the input stream and whether or not you want for the channel to perform live encoding of your stream. You have two options:    

- **None** – Specify this value, if you plan to use an on-premises live encoder which will output multi-bitrate stream. In this case, the incoming stream passed through to the output without any encoding. This is the behavior of a Channel prior to 2.10 release.  For more detailed information about working with channels of this type, see [Working with Channels that Receive Multi-bitrate Live Stream from On-premises Encoders](media-services-manage-channels-overview.md).

- **Standard** (Preview) – Choose this value, if you plan to use Media Services to encode your single bitrate live stream to multi-bitrate stream. 

	Encoding a live stream with Media Services is currently in Preview.

>[AZURE.NOTE]This topic discusses attributes of channels that are enabled to perform live encoding (**Standard** encoding type). For information about working with channels that are not enabled to perform live encoding, see [Working with Channels that Receive Multi-bitrate Live Stream from On-premises Encoders](media-services-manage-channels-overview.md).

The following diagram represents a live streaming workflow where a channel receives a single bitrate stream in one of the following protocols: RTMP, Smooth Streaming, or RTP (MPEG-TS); it then encodes the stream to a multi-bitrate stream. 

![Live workflow][live-overview]

This topic covers the following:

- [Common live streaming scenario](media-services-manage-live-encoder-enabled-channels.md#scenario)
- [Description of a Channel and its related components](media-services-manage-live-encoder-enabled-channels.md#channel)
- [Considerations](media-services-manage-live-encoder-enabled-channels.md#considerations)
- [Tasks related to Live Streaming](media-services-manage-live-encoder-enabled-channels.md#tasks)

##<a id="scenario"></a>Common Live Streaming Scenario

The following steps describe tasks involved in creating common live streaming applications.

1. Connect a video camera to a computer. Launch and configure an on-premises live encoder that can output a single bitrate stream in one of the following protocols: RTMP, Smooth Streaming, or RTP (MPEG-TS). For more information, see [Azure Media Services RTMP Support and Live Encoders](http://go.microsoft.com/fwlink/?LinkId=532824).
	
	This step could also be performed after you create your Channel.

1. Create and start a Channel. 

1. Retrieve the Channel ingest URL. 

	The ingest URL is used by the live encoder to send the stream to the Channel.
1. Retrieve the Channel preview URL. 

	Use this URL to verify that your channel is properly receiving the live stream.

2. Create an asset.
3. If you want for the asset to be dynamically encrypted during playback, do the following: 	
	
	1. 	Create a content key. 
	1. 	Configure the content key's authorization policy.
1. Configure asset delivery policy (used by dynamic packaging and dynamic encryption).
3. Create a program and specify to use the asset that you created.
1. Publish the asset associated with the program by creating an OnDemand locator.  

	Make sure to have at least one streaming reserved unit on the streaming endpoint from which you want to stream content.
1. Start the program when you are ready to start streaming and archiving.
2. Optionally, the live encoder can be signaled to start an advertisement. The advertisement is inserted in the output stream.
1. Stop the program whenever you want to stop streaming and archiving the event.
1. Delete the Program (and optionally delete the asset).   

The [live streaming tasks](media-services-manage-channels-overview.md#tasks) section links to topics that demonstrate how to achieve tasks described above.


##<a id="channel"></a>Channel's input (ingest) configurations

###<a id="Ingest_Protocols"></a>Ingest streaming protocol

If the **Encoder Type** is set to **Standard**, valid options are:

- **RTP** (MPEG-TS): MPEG-2 Transport Stream over RTP.  
- Single bitrate **RTMP**
- Single bitrate **Fragmented MP4** (Smooth Streaming)

For more information, see [Azure Media Services RTMP Support and Live Encoders](http://go.microsoft.com/fwlink/?LinkId=532824).

####RTP (MPEG-TS) - MPEG-2 Transport Stream over RTP.  

Typical use case: 

Use professional broadcasters with on-premises live encoders from vendors like Elemental Technologies, Ericsson, Ateme,  Envivio to send a stream. Often used in conjunction with  an IT department and private networks.

Considerations:

- The use of a single program transport stream (SPTS) input is strongly recommended. The use of multiple language audio tracks, however, is supported
- The video stream should have an average bitrate below 25 Mbps
- The aggregate average bitrate of the audio streams should be below 1 Mbps
- Following are the supported codecs:
	- MPEG-2 / H.262 Video 
		
		- Main Profile (4:2:0)
		- High Profile (4:2:0, 4:2:2)
		- 422 Profile (4:2:0, 4:2:2)

	- MPEG-4 AVC / H.264 Video  
	
		- Baseline, Main, High Profile (8-bit 4:2:0)
		- High 10 Profile (10-bit 4:2:0)
		- High 422 Profile (10-bit 4:2:2)


	- MPEG-2 AAC-LC Audio 
	
		- Mono, Stereo, Surround (5.1, 7.1)
		- MPEG-2 style ADTS packaging

	- Dolby Digital (AC-3) Audio 

		- Mono, Stereo, Surround (5.1, 7.1)

	- MPEG Audio (Layer II and III) 
			
		- Mono, Stereo

- Recommended broadcast encoders include:
	- Ateme AM2102
	- Ericsson AVP2000
	- eVertz 3480
	- Ericsson RX8200
	- Harris Selenio ENC 1
	- Harris Selenio ENC 2
	- AdTec EN-30
	- AdTec EN-91P
	- AdTec EN-100
	- Harmonic ProStream 1000
	- Thor H-2 4HD-EM
	- eVertz 7880 SLKE
	- Cisco Spinnaker
	- Elemental Live

####<a id="single_bitrate_RTMP"></a>Single bitrate RTMP

Considerations:

- The incoming stream cannot have multi-bitrate video, nor can it have multi-track audio
- The video stream should have an average bitrate below 25 Mbps
- The audio stream should have an average bitrate below 1 Mbps
- Following are the supported codecs:

	- MPEG-4 AVC / H.264 Video  
	
		- Baseline, Main, High Profile (8-bit 4:2:0)
		- High 10 Profile (10-bit 4:2:0)
		- High 422 Profile (10-bit 4:2:2)

	- MPEG-2 AAC-LC Audio

		- Mono, Stereo, Surround (5.1, 7.1)
		- 44.1 kHz sampling rate
		- MPEG-2 style ADTS packaging
	
- Recommended encoders include: 

	- Telestream Wirecast, 
	- Flash Media Live Encoder, 
	- Tricaster, etc.

####Single bitrate Fragmented MP4 (Smooth Streaming)

Typical use case: 

Use on-premises live encoders from vendors like Elemental Technologies, Ericsson, Ateme,  Envivio to send the input stream over the open internet to a nearby Azure data center.

Considerations:

Same as for [single bitrate RTMP](media-services-manage-live-encoder-enabled-channels.md#single_bitrate_RTMP).

####Other considerations

- You cannot change the input protocol while the Channel or its associated programs are running. If you require different protocols, you should create separate channels for each input protocol. 
- Maximum resolution for the incoming video stream is 1920x1080, and at most 60 fields/second if interlaced, or 30 frames/second if progressive.


###Ingest URLs (endpoints) 

A Channel provides an input endpoint (ingest URL) that you specify in the live encoder, so the encoder can push streams to your Channels.   

You can get the ingest URLs once you create a Channel. To get these URLs, the Channel does not have to be in the started state. When you are ready to start pushing data into the Channel, it must be in the Started state. Once the Channel starts ingesting data, you can preview your stream through the preview URL.

You have an option of ingesting Fragmented MP4 (Smooth Streaming) live stream over an SSL connection. To ingest over SSL, make sure to update the ingest URL to HTTPS. 

###Allowed IP addresses

You can define the IP addresses that are allowed to publish video to this channel. Allowed IP addresses can be specified as either a single IP address (e.g. ‘10.0.0.1’), an IP range using an IP address and a CIDR subnet mask (e.g. ‘10.0.0.1/22’), or an IP range using an IP address and a dotted decimal subnet mask (e.g. ‘10.0.0.1(255.255.252.0)’). 

If no IP addresses are specified and there is no rule definition then no IP address will be allowed. To allow any IP address, create a rule and set 0.0.0.0/0.


##Channel preview 

###Preview URLs

Channels provide a preview endpoint (preview URL) that you use to preview and validate your stream before further processing and delivery.

You can get the preview URL when you create the channel. To get the URL, the channel does not have to be in the started state. 

Once the Channel starts ingesting data, you can preview your stream.

**Note** Currently the preview stream can only be delivered in Fragmented MP4 (Smooth Streaming) format regardless of the specified input type. You can use the [http://smf.cloudapp.net/healthmonitor](http://smf.cloudapp.net/healthmonitor) player to test the Smooth Stream. You can also use a player hosted in the Azure Management Portal to view your stream.

###Allowed IP Addresses

You can define the IP addresses that are allowed to connect to the preview endpoint. If no IP addresses are specified any IP address will be allowed. Allowed IP addresses can be specified as either a single IP address (e.g. ‘10.0.0.1’), an IP range using an IP address and a CIDR subnet mask (e.g. ‘10.0.0.1/22’), or an IP range using an IP address and a dotted decimal subnet mask (e.g. ‘10.0.0.1(255.255.252.0)’).

##Live encoding settings

This section describes how the settings for the live encoder within the Channel can be adjusted, when the **Encoding Type** of a Channel is set to **Standard**.

###Ad marker source

You can specify the source for ad markers signals. Default value is **Api**, which indicates that the live encoder within the Channel should listen to an asynchronous **Ad Marker API**. 

The other valid option is **Scte35** (allowed only if the ingest streaming protocol is set to RTP (MPEG-TS). When Scte35 is specified, the live encoder will parse SCTE-35 signals from the input RTP (MPEG-TS) stream. 

###CEA 708 Closed Captions

An optional flag which tells the live encoder to ignore any CEA 708 captions data embedded in the incoming video. When the flag is set to false (default), the encoder will detect and re-insert CEA 708 data into the output video streams.

###Video Stream

Optional. Describes the input video stream. If this field is not specified, the default value is used. This setting is allowed only if the input streaming protocol is set to RTP (MPEG-TS).

####Index

A zero-based index that specifies which input video stream should be processed by the live encoder within the Channel. This setting applies only if ingest streaming protocol is RTP (MPEG-TS). 

Default value is zero. It is recommended to send in a single program transport stream (SPTS). If the input stream contains multiple programs, the live encoder parses the Program Map Table (PMT) in the input, identifies the inputs that have a stream type name of MPEG-2 Video or H.264, and arranges them in the order specified in the PMT. The zero-based index is then used to pick up the n-th entry in that arrangement. 

###Audio Stream

Optional. Describes the input audio streams. If this field is not specified, the default values specified apply. This setting is allowed only if the input streaming protocol is set to RTP (MPEG-TS).

####Index

It is recommended to send in a single program transport stream (SPTS). If the input stream contains multiple programs, the live encoder within the Channel parses the Program Map Table (PMT) in the input, identifies the inputs that have a stream type name of MPEG-2 AAC ADTS or AC-3 System-A or AC-3 System-B or MPEG-2 Private PES or MPEG-1 Audio or MPEG-2 Audio, and arranges them in the order specified in the PMT. The zero-based index is then used to pick up the n-th entry in that arrangement.

####Language

The language identifier of the audio stream, conforming to ISO 639-2, such as ENG. If not present, the default is UND (undefined).

There can be up to 8 audio stream sets specified if the input to the Channel is MPEG-2 TS over RTP. However, there can be no two entries with the same value of Index.

###<a id="preset"></a>System Preset

Specifies the preset to be used by the live encoder within this Channel. Currently, the only allowed value is **Default720p** (default).

**Default720p** will encode the video into the following 7 layers.


####Output Audio Stream

Audio is encoded to stereo AAC-LC at 64 kbps, sampling rate of 44.1 kHz. 

##Signaling Advertisements

When your Channel has Live Encoding enabled, you have a component in your pipeline that is processing video, and can manipulate it. You can signal for the Channel to insert slates and/or advertisements into the outgoing adaptive bitrate stream. Slates are still images that you can use to cover up the input live feed in certain cases (for example during a commercial break). Advertising signals, are time-synchronized signals you embed into the outgoing stream to tell the video player to take special action – such as to switch to an advertisement at the appropriate time. See this [blog](https://codesequoia.wordpress.com/2014/02/24/understanding-scte-35/) for an overview of the SCTE-35 signaling mechanism used for this purpose. Below is a typical scenario you could implement in your live event.

1. Have your viewers get a PRE-EVENT image before the event starts.
1. Have your viewers get a POST-EVENT image after the event ends.
1. Have your viewers get an ERROR-EVENT image if there is a problem during the event (for example, power failure in the stadium).
1. Send an AD-BREAK image to hide the live event feed during a commercial break.

The following are the properties you can set when signaling advertisements. 

###Duration

The duration, in seconds, of the commercial break. This has to be a non-zero positive value in order to start the commercial break. When a commercial break is in progress, and the duration is set to zero with the CueId matching the on-going commercial break, then that break is canceled.

###CueId

Unique ID for the commercial break, to be used by downstream application to take appropriate action(s). Needs to be a positive integer. 

###Show slate

Optional. Indicates to the live encoder within the Channel that it needs to switch to the default slate image during the commercial break (and mask the incoming video feed). Default is **false**. 

The image used will be the one specified via the default slate asset Id property at the time of the channel creation. 

##Insert Slate  images

The live encoder within the Channel can be signaled to switch to a slate image. It can also be signaled to end an on-going slate. 

The live encoder can be configured to switch to a slate image and mask the incoming video signal in certain situations – for example, during an ad break. If such a slate is not configured, input video is not masked during that ad break. 

###Duration

The duration of the slate in seconds. This has to be a non-zero positive value in order to start the slate. If there is an on-going slate, and a duration of zero is specified, then that on-going slate will be terminated.

###Insert slate on ad marker

When set to true, this setting configures the live encoder to insert a slate image during an ad break. The default value is true. 

###Default slate Asset Id

Optional. Specifies the Asset Id of the Media Services Asset which contains the slate image. Default is null. 

**Note**: Before creating the Channel, the slate image, of 1920x1080 maximum resolution, in JPEG format, and at most 3 Mbytes in size, should be uploaded as a dedicated asset (no other files should be in this asset). The file name should have a *.jpg extension, and this AssetFile should be marked as the primary file for that asset. This Asset cannot be storage encr2ypted.

If the **default slate Asset Id** is not specified, and **insert slate on ad marker** is set to **true**, a default Azure Media Services image will be used to mask the input stream.



##Channel's programs

A channel is associated with programs that enable you to control the publishing and storage of segments in a live stream. Channels manage Programs. The Channel and Program relationship is very similar to traditional media where a Channel has a constant stream of content and a program is scoped to some timed event on that Channel.

You can specify the number of hours you want to retain the recorded content for the program by setting the **Archive Window** length. This value can be set from a minimum of 5 minutes to a maximum of 25 hours. Archive window length also dictates the maximum amount of time clients can seek back in time from the current live position. Programs can run over the specified amount of time, but content that falls behind the window length is continuously discarded. This value of this property also determines how long the client manifests can grow.

Each program is associated with an Asset which stores the streamed content. An asset is mapped to a blob container in the Azure Storage account and the files in the asset are stored as blobs in that container. To publish the program so your customers can view the stream you must create an OnDemand locator for the associated asset. Having this locator will enable you to build a streaming URL that you can provide to your clients.

A Channel supports up to three concurrently running programs so you can create multiple archives of the same incoming stream. This allows you to publish and archive different parts of an event as needed. For example, your business requirement is to archive 6 hours of a program, but to broadcast only last 10 minutes. To accomplish this, you need to create two concurrently running programs. One program is set to archive 6 hours of the event but the program is not published. The other program is set to archive for 10 minutes and this program is published.

You should not reuse existing programs for new events. Instead, create and start a new program for each event as described in the Programming Live Streaming Applications section.

Start the program when you are ready to start streaming and archiving. Stop the program whenever you want to stop streaming and archiving the event. 

To delete archived content, stop and delete the program and then delete the associated asset. An asset cannot be deleted if it is used by a program; the program must be deleted first. 

Even after you stop and delete the program, the users would be able to stream your archived content as a video on demand, for as long as you do not delete the asset.

If you do want to retain the archived content, but not have it available for streaming, delete the streaming locator.


##Getting a thumbnail preview of a live feed

When Live Encoding is enabled, you can now get a preview of the live feed as it reaches the Channel. This can be a valuable tool to check whether your live feed is actually reaching the Channel. 

##Channel state

The current state of a Channel. Possible values include:

- Stopped. This is the initial state of the Channel after its creation. In this state, the Channel properties can be updated but streaming is not allowed.
- Starting. The Channel is being started. No updates or streaming is allowed during this state. If an error occurs, the Channel returns to the Stopped state.
- Running. The Channel is capable of processing live streams.
- Stopping. The Channel is being stopped. No updates or streaming is allowed during this state.
- Deleting. The Channel is being deleted. No updates or streaming is allowed during this state.


>[AZURE.NOTE] Channel start up can take up to 30 minutes. Channel reset can take up to 5 minutes.



##<a id="Considerations"></a>Considerations

- You cannot change the input protocol while the Channel or its associated programs are running. If you require different protocols, you should create separate channels for each input protocol. 
- You are only billed when your Channel is in running state.
- Every time you reconfigure the live encoder, call the **Reset** method on the channel. Before you reset the channel, you have to stop the program. After you reset the channel, restart the program. 
- A channel can be stopped only when it is in the Running state, and all programs on the channel have been stopped.
- By default you can only add 5 channels to your Media Services account. For more information, see [Quotas and Limitations](media-services-quotas-and-limitations.md).
- You cannot change the input protocol while the Channel or its associated programs are running. If you require different protocols, you should create separate channels for each input protocol.


##<a id="tasks"></a>Tasks related to Live Streaming

###Creating a Media Services account

[Create Azure Media Services Account](media-services-create-account.md).

###Configuring streaming endpoints

For an overview about streaming endpoints and information on how to manage them, see [How to Manage Streaming Endpoints in a Media Services Account](media-services-manage-origins.md)

###Setting up development environment  

Choose **.NET** or **REST API** for your development environment.

[AZURE.INCLUDE [media-services-selector-setup](../includes/media-services-selector-setup.md)]

###Connecting programmatically  

Choose **.NET** or **REST API** to programmatically connect to Azure Media Services.

[AZURE.INCLUDE [media-services-selector-connect](../includes/media-services-selector-connect.md)]

###Creating Channels that perform live encoding from a singe bitrate to adaptive bitrate stream 

Choose **Portal**, **.NET**, **REST API** to see how to create and manage channels and programs.

> [AZURE.SELECTOR]
- [Portal](media-services-portal-creating-live-encoder-enabled-channel.md)
- [.NET](media-services-dotnet-creating-live-encoder-enabled-channel.md)
- [REST](https://msdn.microsoft.com/library/azure/dn783458.aspx

###Protecting assets

If you want to encrypt an asset associate with a program with Advanced Encryption Standard (AES) (using 128-bit encryption keys) or PlayReady DRM, you need to create a content key.

Use **.NET** or **REST API** to create keys.

[AZURE.INCLUDE [media-services-selector-create-contentkey](../includes/media-services-selector-create-contentkey.md)]

Once you create the content key, you can configure key authorization policy using **.NET** or **REST API**.

[AZURE.INCLUDE [media-services-selector-content-key-auth-policy](../includes/media-services-selector-content-key-auth-policy.md)]

###Publishing and delivering assets

**Overview**: 

- [Dynamic Packaging Overview](media-services-dynamic-overview.md)
- [Delivering Content Overview](media-services-deliver-content-overview.md)

Configure asset delivery policy using **.NET** or **REST API**.

[AZURE.INCLUDE [media-services-selector-asset-delivery-policy](../includes/media-services-selector-asset-delivery-policy.md)]

Publish assets (by creating Locators) using **Azure Management Portal** or **.NET**.

[AZURE.INCLUDE [media-services-selector-publish](../includes/media-services-selector-publish.md)]


###Enabling Azure CDN

Media Services supports integration with Azure CDN. For information on how to enable Azure CDN, see [How to Manage Streaming Endpoints in a Media Services Account](media-services-manage-origins.md#enable_cdn).

###Scaling a Media Services account

You can scale **Media Services** by specifying the number of **Streaming Reserved Units** you would like your account to be provisioned with. 

For information about scaling streaming units, see: [How to scale streaming units](media-services-manage-origins.md#scale_streaming_endpoints.md).

##Related topics

[Delivering Live Streaming Events with Azure Media Services](media-services-live-streaming-workflow.md)

[live-overview]: ./media/media-services-overview/media-services-live-streaming-new.png