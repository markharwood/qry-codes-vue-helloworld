<script setup lang="ts">
import { ref, onMounted } from "vue"
import { QryCluster, BitVisualizer } from '@andorsearch/qry-codes-vue'
import { bytesConversion } from "@andorsearch/qry-codes"




let myDocs = ref<any[]>([])

async function loadData() {
  const data = await fetch("/data/news_200.json");
  const json = await data.json()
  json.forEach(doc => {
    // Replace simple string objects loaded in json with binary vectors
    doc["embedding"] = bytesConversion.fromBase64(doc["embedding"])
    myDocs.value.push(doc)
  })
}
onMounted(() => {
  loadData()
});

function doSomethingOnClusterClick(clusterCentroid: Uint8Array) {
  // Query a database here e.g.  https://www.elastic.co/search-labs/blog/bit-vectors-in-elasticsearch
  let hexString = bytesConversion.toHex(clusterCentroid)
  let myElasticsearchQuery = {
    "query": {
      "knn": {
        "field": "embedding",
        "query_vector": hexString
      }
    }
  }
  alert(
    'Typically put application code here to query a vector database with the centroid of this cluster.\n' +
    'Example elasticsearch query for this cluster:\n' +
    JSON.stringify(myElasticsearchQuery, null, 4)
  );
}

</script>

<template>
  <div>
    <h1>Vue binary vector clustering</h1>
    <div v-if="myDocs.length == 0">
      Loading data...
    </div>

    <QryCluster v-if="myDocs.length > 0" :vectors="myDocs.map(doc => doc['embedding'])" :minDocsPerCluster="1">
      <!-- This slot allows the header of each cluster to be rendered. 
          The 'clusterMergedVector' is the average UInt8Array of all elements in the cluster.
          The 'clusterVectorIndices' is an array of numbers representing the elements in the cluster where each
          number is the index of the vector passed in the "vectors" property above
        -->
      <template #clusterHeader="{ clusterMergedVector, clusterVectorIndices }">
        <button class="myClusterHeaderButton" @click="doSomethingOnClusterClick(clusterMergedVector)">
          <div >
            <BitVisualizer :data="clusterMergedVector" :cols="64" :cellSize="1" />
          </div>
          <div v-if="clusterVectorIndices.length > 1" class="headerText">
            {{ clusterVectorIndices.length }} matches
          </div>
          <div v-if="clusterVectorIndices.length == 1" class="headerText">
            1 match
          </div>
        </button>
      </template>

      <!-- This slot allows the content of each element in a cluster to be rendered. 
          The 'vectorIndex' is the index of the vector passed in the "vectors" property above.
        -->
      <template #clusterElement="{ vectorIndex }">
        <div class="article">
          {{ myDocs[vectorIndex]['headline'] }}
        </div>
      </template>
    </QryCluster>

    <div style="text-align: left;">
      <p>Learn more about binary vectors in the client:</p>
      <ul>
        <li>See <a href="https://qry.codes/">QRy codes</a> for a searcher's guide.</li>
        <li>See <a href="https://www.youtube.com/watch?v=sJU_8mtzH7Y">this video</a> for a developer's guide</li>
      </ul>
    </div>

  </div>
</template>

<style>
.headerText {
  font-size: x-small;
  font-weight: bold;
}

.article {
  font-size: x-small;
  border-radius: 5px;
  text-align: left;
  padding: 10px;
  max-width: 200px;
  border-style: solid;
  border-color: #f1f1f1;
  background-color: #efefef;
}

.qry-codes-cluster {
  font-size: x-small;
  border-radius: 5px;
  text-align: left;
  padding: 10px;
  max-width: 200px;
  border-style: solid;
  border-color: #f1f1f1;
}

.qry-codes-cluster:hover {
  background-color: lightgray;
}
</style>
