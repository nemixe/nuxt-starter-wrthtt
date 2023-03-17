<template>
  <div>
    <b id="title"> Show PDF Hamstring Flexibility </b>
    <div ref="pdf-container" id="pdf-container"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pdfjsLib: null,
      scale: 1,
    };
  },
  mounted() {
    import('pdfjs-dist/build/pdf').then((pdfMod) => {
      this.pdfjsLib = pdfMod;
      this.pdfjsLib.GlobalWorkerOptions.workerSrc =
        'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.worker.min.js';
      this.loadPdfFromUrl(
        'https://assets.stanwith.me/live/msc/27112/iufy7/mbd%20hamstring%20flexibility.pdf'
      );
    });
  },
  methods: {
    async initPdfJs() {
      try {
        const pdfJS = await import('pdfjs-dist/build/pdf');
        pdfJS.GlobalWorkerOptions.workerSrc =
          'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.worker.min.js';

        const pdf = await pdfJS.getDocument(
          'https://assets.stanwith.me/live/msc/27112/iufy7/mbd%20hamstring%20flexibility.pdf'
        ).promise;

        const page = await pdf.getPage(1);
        const viewport = page.getViewport({ scale: 1.5 });

        const canvas = this.$refs['pdf-container'];
        const canvasContext = canvas.getContext('2d');
        canvas.height = viewport.height;
        canvas.width = viewport.width;

        // Render PDF page into canvas context.
        const renderContext = { canvasContext, viewport };
        page.render(renderContext);
      } catch (err) {
        console.error(err);
      }
    },
    loadPdfFromUrl(url) {
      //Read PDF from URL.
      this.pdfjsLib.getDocument(url).promise.then((pdfDoc) => {
        //Reference the Container DIV.
        var pdf_container = this.$refs['pdf-container'];
        pdf_container.style.display = 'block';
        pdf_container.style.height = this.isMobile() ? '1200px' : '820px';

        //Loop and render all pages.
        for (var i = 1; i <= pdfDoc.numPages; i++) {
          this.renderPdfPage(pdfDoc, pdf_container, i);
        }
      });
    },
    renderPdfPage(pdfDoc, pdf_container, num) {
      pdfDoc.getPage(num).then((page) => {
        //Create Canvas element and append to the Container DIV.
        var canvas = document.createElement('canvas');
        canvas.id = 'pdf-' + num;
        const ctx = canvas.getContext('2d');
        const resolution = this.isMobile() ? 1.5 : 1;
        pdf_container.appendChild(canvas);

        //Create and add empty DIV to add SPACE between pages.
        var spacer = document.createElement('div');
        spacer.style.height = '20px';
        pdf_container.appendChild(spacer);

        //Set the Canvas dimensions using ViewPort and Scale.
        var viewport = page.getViewport({ scale: this.scale });
        canvas.height = resolution * viewport.height;
        canvas.width = resolution * viewport.width;

        //Render the PDF page.
        var renderContext = {
          canvasContext: ctx,
          viewport: viewport,
          transform: [resolution, 0, 0, resolution, 0, 0],
        };

        page.render(renderContext);
      });
    },
    isMobile() {
      var r = new RegExp(
        'Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini'
      );
      return r.test(navigator.userAgent);
    },
  },
};
</script>

<style type="text/css">
#pdf-container {
  background: #ccc;
  text-align: center;
  padding: 5px;
  overflow: auto;
}
#title {
  margin-bottom: '80px';
}
</style>
