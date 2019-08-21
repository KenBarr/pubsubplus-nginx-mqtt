# pubsub+ NGiNX Plus MQTT QoS1 consumer solution
Solution based on nginx keyval to always get MQTT consumer back to their QoS1 subscription in a horizontally scaled broker solution
The fondations of this solution is applicable to getting any client back to a statefull service as that service horizontally scales

Current state of the art uses consistent hash to get consumer back to thier Q0S1 subscription on reconnect.  This works until the number of backend MQTT brokers scales up causing a new modulo in the hash.

Solve this problem using NGiNX Plus keyval so the load balancers remember where every consumer originally connects and gets them back there on reconnect.  Connection loading would look like this:
 - all connections go to initial broker
 - once initial broker has reached,(or nearly reached), connection limit; scale up new broker and dircect all new connections to it
 - continue cycle of creating brokers and moving new connections to new broker, while previously known connections are directed toward their original broker.
 - this does not take into accont scale down yet. 

Pre-requisits
NGiNX Plus as it has built in module: ngx_stream_keyval_module
Enable REST POST and PATCH admin access:  dav_methods PUT DELETE MKCOL COPY MOVE;
Seed the initial backend broker after starting nginx: curl -X POST http://localhost:80/api/4/stream/keyvals/mqtt_seed -d '{"default":"XX.XX.XX.XX:1883"}'

After each scale up event patch the new seed IP of the latest broker: curl -X PATCH http://localhost:80/api/4/stream/keyvals/mqtt_seed -d '{"default":"YY.YY.YY.YY:1883"}'

There currently does not seem to be a way to force a sync of the new seed.
