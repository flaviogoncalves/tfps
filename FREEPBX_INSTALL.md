# FreePBX Integration Guide

FPS is a simple system to protect your PBX from fraudulent calls. It is very easy to integrate to your PBX. After creating your account and setting your calling policies, you will need to configure your FreePBX (Elastix, Trixbox and others). FPS works like a SIP Provider, but instead of completing your call, it redirects your call back with a three character prefix. After receiving this prefix, you can decide what to do in your dial plan. Below a series of steps required to implement the FPS on FreePBX.

## Method #1 Redirect

To integrate FPS with FreePBX you need:

1 - Add a trunk to FPS
2 -	Add a trunk to PSTN
3 - Add the FPS code to extensions_custom.conf

Step 1: Add a trunk to FPS

Name the trunk “fps” and fill the PEER details replacing <username> and <password> by your accounting data, replase server by your own server and port by the port of the TFPS server, usually 5060. Leave the USER details empty.

![Imagem1](https://user-images.githubusercontent.com/4958202/129276346-c4071044-2020-42fb-8475-6af7af7e5fd5.png)
 
Step 2: Add the FPS instructions to /etc/asterisk/extensions_custom.conf

```
[from-internal-custom]
exten => h,1,Hangup()
include => agentlogin
include => conferences
include => calendar-event
include => weather-wakeup
include => tfpsdial

[tfpsdial]
exten=_011[1-9].,1,set(GROUP()=fps)
same=>n,set(ncalls=${GROUP_COUNT(fps)})
same=>n,set(_original=${EXTEN})
same=>n,SIPAddHeader(P-tfps: ${CALLERID(num)};${SIPDOMAIN};${CHANNEL(recvip)};${CHANNEL(useragent)};${ncalls})
same=>n,set(_original=${EXTEN})
same=>n,dial(SIP/tfps/${FILTER(0-9,${EXTEN:2})})
same=>n,playback(ss-noservice);

[tfps]
;For calls approved, they will receive a redirect and get to the tfps context
exten=_A.,1,Answer()
same=>n,set(OUTBOUND_TRUNK=SIP/International) ; Set here your PSTN trunk 
same=>n,Dial(${OUTBOUND_TRUNK}/${original})
same=>n,hangup(16)

Response Codes
A00 – Call Approved
``` 
### Method 2 Failover

Step 1: Add a trunk to FPS

Name the trunk “fps” and fill the PEER details replacing <username> and <password> by your accounting data, replase server by your own server and port by the port of the TFPS server, usually 5060. Leave the USER details empty.

![Imagem1](https://user-images.githubusercontent.com/4958202/129276346-c4071044-2020-42fb-8475-6af7af7e5fd5.png)

Step 2: Add the FPS instructions to /etc/asterisk/extensions_custom.conf
 
```
[from-internal-custom]
exten => h,1,Hangup()
include => agentlogin
include => conferences
include => calendar-event
include => weather-wakeup
include => tfpsdial

[tfpsdial]
exten=_011[1-9].,1,set(GROUP()=fps)
same=>n,set(ncalls=${GROUP_COUNT(fps)})
same=>n,set(_original=${EXTEN})
same=>n,SIPAddHeader(P-tfps: ${CALLERID(num)};${SIPDOMAIN};${CHANNEL(recvip)};${CHANNEL(useragent)};${ncalls})
same=>n,dial(SIP/tfps/${FILTER(0-9,${EXTEN:2})})
same=>n,goto(${DIALSTATUS})
same=>n,hangup()
same=>n(BUSY),playback(ss-noservice)
same=>n,hangup()
same=>n(CONGESTION),
same=>n,Dial(SIP/provider/${EXTEN})
 
 
DISCLAIMER
No service can guarantee 100% that you will not be a victim of fraud. We can remove 99.999% of all attacks using our system, but it is wise to apply additional measures. We strongly advise you to, beyond installing this system, take other measures not limited to:
1.	Do not allow Internet Access to your PBX web interface and or SSH.
2.	Check your CDRs regularly for strange or fraudulent calls.
3.	Prefer limited prepaid SIP trunking for International calls instead of post-paid unlimited TDM trunks.
4.	Use strong passwords always.

TECH-SUPPORT
Please send any tech support requests to info@sippulse.com
