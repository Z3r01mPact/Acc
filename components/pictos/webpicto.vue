<template>
  <div @click="downloadImg(webpicto.download)">
    <b-tooltip
      type="is-dark"
      position="is-bottom"
      is-small
      multilined
      :delay="1000"
      :triggers="['hover']"
    >
      <img id="svg" style="aspect-ratio: 1/1; max-height: 13vh;" loading="lazy" :src="webpicto.src"></img>
      
      <template v-slot:content>
        {{ webpicto.source }}/{{ webpicto.author }}
      </template>
    </b-tooltip>
    <b>{{ webpicto.title }}</b>
  </div>
</template>

<script >
export default {
  name: "Webpicto",

  props: {
    webpicto: {
      type: Object,
      required: true,
    },
  },
  methods: {
    async downloadImg(src) {
      this.show = 1;
      let response;
      response = await fetch(src);
      response = await response.blob();
      if (src.includes(".svg")) {
        let promise = new Promise((resolve, reject) => {
          let url = window.webkitURL.createObjectURL(response);
          let canvas = document.createElement("canvas");
          canvas.width = 851;
          canvas.height = 851;
          let ctx = canvas.getContext("2d");
          let img = new Image();
          img.loading;
          img.onload = function () {
            ctx.drawImage(this, 0, 0);
            let styled = document.createElement("canvas");
            styled.width = canvas.width;
            styled.height = canvas.height;
            const styledCtx = styled.getContext("2d");
            styledCtx.save();
            styledCtx.fillStyle = "#ffffff";
            styledCtx.fillRect(0, 0, canvas.width, canvas.height);
            styledCtx.strokeRect(0, 0, canvas.width, canvas.height);
            styledCtx.restore();
            styledCtx.drawImage(canvas, 0, 0);
            canvas = styled;
            window.webkitURL.revokeObjectURL(url);
            const dataURI = canvas.toDataURL();
            const byteString = atob(dataURI.split(",")[1]);
            const mimeString = dataURI
              .split(",")[0]
              .split(":")[1]
              .split(";")[0];
            const ab = new ArrayBuffer(byteString.length);
            const ia = new Uint8Array(ab);
            for (let i = 0; i < byteString.length; i++) {
              ia[i] = byteString.charCodeAt(i);
            }
            const blob = new Blob([ab], { type: mimeString });
            resolve(blob);
          };
          img.src = url;
        });
        const pngblob = await promise.then((value) => {
          return value;
        });
        this.createFileFromBlob(pngblob, "png", src);
      } else {
        this.createFileFromBlob(response, "jpg", src);
      }
    },
    createFileFromBlob(blob, type, url) {
      const file = new File(
        [blob],
        `${this.webpicto.title.replaceAll(
          /[&\/\\#, +()$~%.'":*?<>{}]/g,
          "-"
        )}.${type}`,
        { type: blob.type }
      );
      file.url = url;
      this.$emit("uploadfile", file);
    },
  },
};
</script>
