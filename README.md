# Twilio-Asterisk
Asterisk SIP Trunking

Install Asterisk asterisk-certified-16.8-current.tar.gz

1. Follow instructions to set up Asterisk

- https://wiki.asterisk.org/wiki/display/AST/Installing+Asterisk
- or YouTube Tutorial
https://www.youtube.com/watch?v=jMQfSsO1da4&list=PLnzEbgyK52Gu9fdVDHburrsG3KBIntXFK&index=1

This was recorded in 2015 and most examples are based in chan_sip has been deprecated, but still a good tutorial to get the sense of setting up Asterisk.
Examples in this file are using chan_pjsip.

2. Zoiper 5 is used for this project
Download Zoiper5_5.5.5_x86_64.tar.xz from
https://www.zoiper.com/en/voip-softphone/download/current


Setup zopier with user name and login 

<a href="https://www.twilio.com">
  <img src="zopier-setup.png" alt="zopier-setup" width="250" />
</a>

3. Setup Twilio SIP trunk in Twilio account following instructions:

https://www.twilio.com/docs/documents/61/TwilioElasticSIPTrunking-AsteriskPBX-Configuration-Guide-Version2-1-FINAL-09012018.pdf


Noted that using SIP registration simplified the setup without firewall setting as the Asterisk setup above. 

Bold text is subject to setup changes. 

pjsip.conf

[transport-udp]
type=transport
protocol=udp
bind=0.0.0.0

pjsip_wizard.conf

user_defaults](!)
type = wizard
accepts_registrations = yes
sends_registrations = no
accepts_auth = yes
sends_auth = no
has_hint = yes
hint_context = employee
hint_application = Gosub(stdexten,${EXTEN},1(${HINT}))
endpoint/context = employee
endpoint/allow_subscribe = yes
endpoint/allow = !all,ulaw,gsm,g722
endpoint/direct_media = yes
endpoint/force_rport = yes
endpoint/disable_direct_media_on_nat = yes
endpoint/direct_media_method = invite
endpoint/ice_support = yes
endpoint/moh_suggest = default
endpoint/send_rpid = yes
endpoint/rewrite_contact = yes
endpoint/send_pai = yes
endpoint/allow_transfer = yes
endpoint/trust_id_inbound = yesendpoint/device_state_busy_at = 1
endpoint/trust_id_outbound = yes
endpoint/send_diversion = yes
aor/qualify_frequency = 30
aor/authenticate_qualify = no
aor/max_contacts = 1
aor/remove_existing = yes
aor/minimum_expiration = 30
aor/support_path = yes
phoneprov/PROFILE = profile1

[joe](user_defaults)
hint_exten = 6001
inbound_auth/username = joe
inbound_auth/password = joepassword

[jane](user_defaults)
hint_exten = 6002
inbound_auth/username = jane
inbound_auth/password = janepassword

[trunk_defaults](!)
type = wizard
endpoint/allow = !all,ulaw,gsm
endpoint/t38_udptl=no
endpoint/t38_udptl_ec=none
endpoint/fax_detect=no
endpoint/trust_id_inbound=no
endpoint/t38_udptl_nat=no
endpoint/direct_media=no
endpoint/rewrite_contact=yes
endpoint/rtp_symmetric=yes
endpoint/dtmf_mode=rfc4733
endpoint/allow_subscribe = no
aor/qualify_frequency = 60

[twiliotrunk](trunk_defaults)
sends_auth = yes
sends_registrations = yes
remote_hosts = asterisk-twillio.pstn.twilio.com
outbound_auth/username = asteriskPBX
outbound_auth/password = Twilio1asterisk
endpoint/context = siptrunk
aor/qualify_frequency = 60

extensions.conf

[siptrunk]

; Assume the acquired Twilio number is +14101234567 
; DID of Joe's extension when setting Twilio SIP trunk number to Joe's extension
;exten => +14101234567,1,Dial(PJSIP/joe)

; Set the Twilio SIP trunk number to the main switching board 100 for extension dialing
exten => +14101234567,1,Goto(employee,100,1)

; Out-bound Dialing
; Matching of any PSTN US number
; Twilio SIP trunking will authenticate terminating to Twilio calls based on
; setting of CALLERID to Twilio SIP trunking number

exten => _NXXNXXXXXX,1,Set(CALLERID(all)="Eevee" <+14101234567>)
same = n,Dial(PJSIP/+1${EXTEN}@twiliotrunk)
same = n,(end), Hangup()

[employee]

include => siptrunk

; Inbound extension dialing

exten => 100,1,Answer()
same => wait(2)
same => n,Background(dir-pls-enter&extension&dir-pls-enter&extension)
same => n,WaitExten(10)
same => n,dial(PJSIP/${Exten})

exten => i,1,Playback(invalid)
same => n,Goto(employee,100,1)

exten => t,1,Playback(vm-goodbye)
same => n,Hangup()

exten = 6001,1,dial(PJSIP/joe)

exten = 6002,1,dial(PJSIP/jane)
