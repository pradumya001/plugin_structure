# plugin_structure

//"use strict";
(function () {
  window.OfscPlugin = function () {
    this.init = function () {
      window.addEventListener("message", this._messageListener.bind(this), false);
      this.sendPostMessageData({
        apiVersion: 1,
        method: "ready",
        sendInitData: true,
        showHeader: true,
        enableBackButton: true,
      });
    };

    this.getOrigin = function (url) {
      if (!url) return "";
      const urlParts = url.indexOf("://") > -1 ? url.split("/") : [url];
      return `https://${urlParts[2] || urlParts[0]}`;
    };

    this.open = function (data) {

          //plugin codes goes here------------
    }

    this.sendPostMessageData = function (data) {
      const origin = this.getOrigin(document.referrer);
      if (!origin) {
        console.error("Invalid origin:", document.referrer);
        return;
      }
      //console.log("Sent>>", JSON.stringify(data, undefined, "\t"));
      parent.postMessage(JSON.stringify(data), origin);
    };

    this._messageListener = function (event) {
      try {
        const data = JSON.parse(event.data);
        //console.log("Resource data recieved >>", JSON.stringify(data.resource, undefined, "\t"));
        
        switch (data.method) {
          case "init":
            localStorage.setItem("simplePlugin", JSON.stringify(data.attributeDescription));
            this.sendPostMessageData({
              apiVersion: 1,
              method: "initEnd",
            });
            break;
          case "open":
            this.open(data);
            break;
          case "callProcedureResult":
            console.log("callProcedureResult received");
            break;
          case "updateResult":
            console.log("UpdateResult method called");
            break;
          case "error":
            console.error("Error method called");
            alert("Error: " + JSON.stringify(data.error));
            break;
          case "close":
            this.sendPostMessageData({
              apiVersion: 1,
              method: "close",
            });
            break;
        }
      } catch (e) {
        console.error("Error parsing message:", e);
      }
    };
  };

  (function($){
    $(document).ready(function(){
        var plugin = new OfscPlugin(true)
        plugin.init();
    });
  })

  window.addEventListener("load", function () {
    const plugin = new OfscPlugin();
    plugin.init();
  });
})();
