# pubsubplus-nginx-mqtt
Solution based on nginx keyval to always get MQTT consumer back to their QoS1 subscription in a horizontally scaled broker solution

Current state of the art uses consistent hash to get consumer back to thier Q0S1 subscription on reconnect.  This works until the number of brokers scales up causing a new modulo in the hash.

Trying to solve this problem using NGiNX Plus keyval so the load balancers remember where every consumer originally connects and gets them back there on reconnect.  Connection loading would lookm like this:
 - all connections go to initial broker
 - once initial broker has reached,(or nearly reached), connection limit scale up new broker and move all new connections to it
 - continue cycle of creating brokers and moving new connections to it while not re-directing older known connections.
 - this does not take into accont scale down yet. 
