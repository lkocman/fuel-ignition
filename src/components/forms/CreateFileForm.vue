<template>
  <div class="createfiles">
    <FormKit
      :name="formKey('path')"
      label="File Path (required)"
      placeholder="path on the filesystem"
      validation="required"
      validation-behavior="live"
      value="/etc/someconfig"
      help="The absolute path to the file"
    />

    <FormKit
      :name="formKey('overwrite')"
      label="Overwrite"
      placeholder="path on the filesystem"
      validation="required"
      type="checkbox"
      validation-behavior="live"
      help="Whether to delete preexisting nodes at the path"
    />

    <FormKit
      :name="formKey('mode')"
      label="File Mode (required)"
      type="number"
      value="420"
      validation="required"
      validation-behavior="live"
      help="The file's permission mode. Note that the mode must be properly specified as a decimal value (i.e. 0644 -> 420)."
    />

    <div class="source">
      <FormKit
        type="select"
        :name="formKey('source_type')"
        v-model="sourceType"
        label="Scheme for file contents url (use data for plain text)"
        help="If source is omitted and a regular file already exists at the path, Ignition will do nothing. If source is omitted and no file exists, an empty file will be created."
        :options="['data', 'https', 'http', 'tftp', 's3', 'gs', 'omit']"
      />
    </div>

    <div v-if="sourceType === 'data'" class="data">
      <FormKit
        :name="formKey('data_content')"
        label="File Content Data (required)"
        placeholder="write the file content here, spaces, newlines etc. are preserved"
        type="textarea"
        validation="required"
        validation-behavior="live"
        help="Leaving this empty will create an empty file"
      />
    </div>

    <div v-if="sourceType === 'https'" class="https">
      <FormKit
        :name="formKey('https_content')"
        label="HTTPS Url (required)"
        placeholder="the URL of the file contents"
        type="text"
        validation="required|url"
        validation-behavior="live"
      />
    </div>

    <div v-if="sourceType === 'http'" class="http">
      <FormKit
        :name="formKey('http_content')"
        label="HTTP Url (required)"
        placeholder="the URL of the file contents"
        type="text"
        validation="required|url"
        validation-behavior="live"
      />

      <FormKit
        :name="formKey('http_verification')"
        label="Verification Hash (optional)"
        placeholder="e.g. sha512-012345678.."
        type="text"
        validation="optional"
        validation-behavior="live"
        help="The hash of the contents, in the form <type>-<value> where type is either sha512 or sha256."
      />
    </div>

    <div
      v-if="
        !sourceType.includes('http') &&
        sourceType !== 'data' &&
        sourceType !== 'omit'
      "
      class="tftp-s3-gs"
    >
      <FormKit
        :name="formKey('tftp_s3_gs_content')"
        :label="sourceType.toUpperCase() + ' Url (required)'"
        placeholder="the URL of the file contents"
        type="text"
        validation="required"
        validation-behavior="live"
      />
    </div>
  </div>
</template>

<script>
import Utils from "../../utils/utils.js";
import { ref } from "vue";
const formPrefix = "create_file";

export default {
  setup: () => {
    const uid = Utils.uid();
    const sourceType = ref("data");
    return {
      sourceType,
      formKey: (key) => Utils.getFormKey(formPrefix, key, uid),
    };
  },

  methods: {
    encodeToIgn: function (json, formData) {
      const formValue = (key, uid) =>
        Utils.getFormValue(formPrefix, formData, key, uid);

      const keyPrefix = formPrefix + "_path_";
      Object.keys(formData)
        .filter((x) => x.includes(keyPrefix))
        .map((key) => key.replace("create_file_path_", ""))
        .forEach((id) => {
          json.storage =
            "storage" in json
              ? json.storage
              : {
                  files: [],
                };

          let content;
          let fileObject = {};

          console.log(formValue("source_type", id));
          switch (formValue("source_type", id)) {
            case "data":
              content =
                "data:text/plain;charset=utf-8;," +
                encodeURIComponent(formValue("data_content", id));
              break;

            case "https":
              content = formValue("https_content", id);
              break;

            case "http":
              content = formValue("http_content", id);
              let verification = formValue("http_verification", id);
              if (verification)
                fileObject.verification = { hash: verification };
              break;

            case "tftp":
            case "s3":
            case "gs":
              content = formValue("tftp_s3_gs_content", id);
              break;
          }

          // merging the two objects, in case verification was written to fileObject
          json.storage.files.push(
            Object.assign(
              {
                path: formValue("path", id),
                mode: formValue("mode", id),
                overwrite: formValue("overwrite", id),
                contents: {
                  source: content,
                },
              },
              fileObject
            )
          );
        });
    },
  },
};
</script>
