# Deleting XDCR replications automation
Couchbase url_encoded_replication_id API creation and deletion.
I was looking a way to create and delete XDCR replications automatically via API but I didn't find any step-by-step doc.
First of all , we need to place uuid generate by API XDCR replication creation in a file while generating the replication.
If I take the couchbase sample and add output file that we call uuid.json  :

curl -v -X POST -u Administrator:password1 \
  http://10.4.2.4:8091/controller/createReplication \
  -d 'fromBucket=beer-sample' \
  -d 'toCluster=remote1' \
  -d 'toBucket=remote_beer' \
  -d 'replicationType=continuous' \
  -d 'type=capi' \
  -o uuid.json
  
  We can see that the file uuid.json contains the following info :
  {
  "id": "9eee38236f3bf28406920213d93981a3/beer-sample/remote_beer"
  }
  
  Now , if we want to delete this XDCR replications , we need to install jq so we can easily parse this generated file .
  
Install JQ:  
wget https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64 -O jq
chmod 755 jq
sudo mv jq /usr/local/bin/

delete process :
As explain in couchbase documentation : Replication ID needs to be URL-encoded before it can be used in the delete replication URL.
So :
cat uuid.json | jq -r '.id | @uri' 
command will return url_encoded_replication_id
9eee38236f3bf28406920213d93981a3%2fbeer-sample%2fremote_beer
Create a variable that will contain this url_encoded_replication_id from file created
 uuid=$( cat uuid.json | jq -r '.id | @uri')
 add it at the end of curl 
 curl -u Administrator:password1  \
  http://10.4.2.4:8091/controller/cancelXDCR/ \
  $uuid \
  -X DELETE 
  The process of creation and deletion is completely automated.
  
  a Bientot

