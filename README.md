# Intermediate-Bridge
Bridge local broker to remote broker using an "Intermmediate" broker with TSL
INTERMEDIATE BRIDGE
With the non-secured connection, a bridge was created and it was enough to forward all incoming
messages to port 8883 at the Public IP interface, to the Internal IP of our OpenStack instance
(Figure-26). But TLS creates new challenges and a secured connection can’t be created with the
same configuration.
The SSL Handshaking process fails in our configuration, and this makes impossible to authenticate
the certificates. In consequence, the communication process can’t go beyond the request and is
39
timed-out. A simple solution to this problem is creating what we will call “Intermediate Broker”.
Therefore, instead of forwarding all incoming communications on port 8883, we can install a
Mosquitto Broker in the in the VM with the IP exposed to internet. Then, create another bridge
with our Remote Broker. A full secured connection is created from the Local Broker to the Remote
Broker thanks to the “Intermediate Broker”.
The Intermediate Broker configuration is similar to the Remote Broker’s, and requires the
CA authority files to create its own server and client certificates (using the generate-CA.sh script).
Figure-28. Unsecured MQTT Bridge.
Figure-29. Secured MQTT Bridge.
After installing TLS authentication we have a more secured system, where Brokers are
authenticated by username-password combination plus certificates. Also, the communications are
done by secured channels and using encryption methods. Some processing load could be saved
once the Intermediate Broker is reached, and using unsecured communications inside the Remote
Broker’s LAN seems to be reasonable.
Although, encrypting the MQQT messages inside the LAN gives additional protection against
breaches in the Intermediate Broker’s security. Also, because we could be using a shared set of
40
hardware and virtual resources running other applications, encrypting our messages protects them
from vulnerabilities of other processes using the same resources.
The Intermediate Broker could be used to distribute topics among many Remote Brokers running
in different OpenStack Instances. For example (Figure-29), the Intermediate broker could have
bridges to each Remote Broker, and for each one a different set of topics will be delivered. The
Intermediate Broker can easily segregate topics in a Cloud for different Mosquitto services or
customers that don’t want to share computing or storage resources with each other.
Figure-30. Intermediate Broker.
The mosquitto.conf file for the Intermediate Broker (and many bridged Remote Brokers) should
include the following settings:
cafile /pathtocertificates/ca.crt
certfile /pathtocertificates/server.crt
keyfile /pathtocertificates/server.key
bridge Broker1
remote_cafile /pathtocertificates/ca1.crt
remote_certfile /pathtocertificates/cliente11.crt
remote_keyfile /pathtocertificates/client11.key
remote_username user1
remote_password password1
address 10.0.1.8:8883
topic outTopic/broker1 both
bridge Broker2
remote_cafile /pathtocertificates/ca2.crt
remote_certfile /pathtocertificates/cliente21.crt
remote_keyfile /pathtocertificates/client21.key
remote_username user2
41
remote_password password2
address 10.0.1.9:8883
topic outTopic/broker2 both
……
Code-5. Intermediate Broker mosquitto.conf file.
Where each Bridgen to Brokern has a different user-password combination, Certificate
Authority and keys as well.24 Before the implementation of the Intermediate Broker, we were
only able to forward all traffic on port 8883 to one IP address.
