<template>
  <button
    class="btn btn-primary mb-4"
    style="width: 100%"
    @click="convertAndDownload()"
  >
    Convert and Download
  </button>

  <div v-if="loading" class="sk-folding-cube">
    <div class="sk-cube1 sk-cube"></div>
    <div class="sk-cube2 sk-cube"></div>
    <div class="sk-cube4 sk-cube"></div>
    <div class="sk-cube3 sk-cube"></div>
  </div>
</template>

<script setup>
import { ref } from "vue";
const props = defineProps(["ignJson"]);
var loading = ref(false);

String.prototype.replaceAt = function (index, replacement) {
  return (
    this.substring(0, index) +
    replacement +
    this.substring(index + replacement.length)
  );
};

function bufferToHex(buffer) {
  return [...new Uint8Array(buffer)]
    .map((b) => b.toString(16).padStart(2, "0"))
    .join("");
}

function strToHex(str) {
  var result = "";
  for (var i = 0; i < str.length; i++) {
    result += str.charCodeAt(i).toString(16);
  }
  return result;
}

async function convertAndDownload() {
  this.loading = true;
  console.log("loading: " + this.loading);

  let hexJson = strToHex(JSON.stringify(props.ignJson));
  let jsonByteSize = JSON.stringify(props.ignJson).length;

  console.log(jsonByteSize.toString(16));
  console.log(jsonByteSize.toString(16).slice(2)[1]);

  // let hexJsonByteSize =
  //   jsonByteSize < 255
  //     ? jsonByteSize.toString(16)
  //     : jsonByteSize.toString(16).slice(2)[1] +
  //       jsonByteSize.toString(16).slice(2)[0];


  let hexJsonByteSize =
    jsonByteSize < 255
      ? jsonByteSize.toString(16)
      : "4E2000"; // pad to even number and then flip in the middle
  console.log("flipped tuple: " + (jsonByteSize > 255));
  console.log(hexJsonByteSize);

  let file = await fetch("src/assets/template/ignition.img").then((response) =>
    response.blob()
  );

  console.log(file);

  console.log("jsonSize(dec): " + JSON.stringify(props.ignJson).length);
  console.log("jsonSize(hex): " + hexJsonByteSize);

  let decimalOffset = 43068 * 2; // 1 byte = 8 bit = 2^8 = 256 = 0xFF
  file.arrayBuffer().then((buffer) => {
    let hex = bufferToHex(buffer);
    console.log(new Blob([hex]).size);
    console.log(hex.slice(decimalOffset, decimalOffset + 4));

    let cleanedHex = hex
      .replace("6c6f72656d697073756d", hexJson)
      .replaceAt(decimalOffset, hexJsonByteSize); // replace 'loremipsum' with 'helloworld', same length

    console.log(cleanedHex.slice(decimalOffset, decimalOffset + 4));

    if (cleanedHex.length % 2) {
      alert("Error: cleaned hex string length is odd.");
      return;
    }

    var binary = new Array();
    for (var i = 0; i < cleanedHex.length / 2; i++) {
      var h = cleanedHex.substr(i * 2, 2);
      binary[i] = parseInt(h, 16);
    }

    this.loading = false;

    var byteArray = new Uint8Array(binary);
    var a = window.document.createElement("a");

    a.href = window.URL.createObjectURL(
      new Blob([byteArray], { type: "application/octet-stream" })
    );
    a.download =
      "ignition-" +
      alphabet[Math.floor(Math.random() * alphabet.length)].toLowerCase() +
      ".img";

    // Append anchor to body.
    document.body.appendChild(a);
    a.click();

    // Remove anchor from body
    document.body.removeChild(a);
  });
}

let alphabet = [
  "Alfa",
  "Bravo",
  "Charlie",
  "Delta",
  "Echo",
  "Foxtrot",
  "Golf",
  "Hotel",
  "India",
  "Juliett",
  "Kilo",
  "Lima",
  "Mike",
  "November",
  "Oscar",
  "Papa",
  "Quebec",
  "Romeo",
  "Sierra",
  "Tango",
  "Uniform",
  "Victor",
  "Whiskey",
  "X-ray",
  "Yankee",
  "Zulu",
  "Nico",
];
</script>
