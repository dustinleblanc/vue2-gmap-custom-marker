<template>
  <div>
    <slot></slot>
  </div>
</template>
<script>
import Vue from 'vue';
import * as VueGoogleMaps from 'vue2-google-maps';
export default {
  mixins: [VueGoogleMaps.MapElementMixin],
  props: ['marker', 'onClick'],
  watch: {
    marker: function (val) {
      this.$overlay.draw();
    },
  },
  data () {
    return {
      clickListener: {remove () {}}
    }
  },
  computed: {

    bounds () {
      var lat = parseFloat(this.marker.latitude);
      var lng = parseFloat(this.marker.longitude);
      return new google.maps.LatLngBounds(
        new google.maps.LatLng(lat, lng),
        new google.maps.LatLng(lat + 0.01, lng + 0.01));
    },
    position () {
      var self = this;
      return {
        lat () {
          return parseFloat(self.marker.latitude);
        },
        lng () {
          return parseFloat(self.marker.longitude);
        }
      }
    }
  },
  deferredReady: function() {
    this.initOverlay();
  },
  destroyed: function() {
    this.$overlay.setMap(null);
    if (self.$el) {
      while (self.$el.firstChild) {
        self.$el.removeChild(self.$el.firstChild);
      }
    }
    this.$overlay = undefined;
    this.clickListener.remove();
  },
  methods: {
    initOverlay (createElement) {

      var self = this;
      Overlay.prototype = new google.maps.OverlayView();

      /** @constructor */
      function Overlay(map) {
        // Initialize all properties.
        this.map_ = map;
        // Explicitly call setMap on this overlay.
        this.setMap(map);
      }


      Overlay.prototype.setPosition = function(position) {
        position = position || this.position;
        var _this = this;
        if (this.getProjection() && typeof position.lng == 'function') {
          var setPosition = function() {
            if (!_this.getProjection()) { return; }
            var posPixel = _this.getProjection().fromLatLngToDivPixel(position);
            var x = Math.round(posPixel.x - (_this._div.offsetWidth/2));
            var y = Math.round(posPixel.y - _this._div.offsetHeight - 10); // 10px for anchor
            _this._div.style.left = x + "px";
            _this._div.style.top = y + "px";
            _this._div.style.visibility = "visible";
          };
          if (_this._div.offsetWidth && _this._div.offsetHeight) {
            setPosition();
          } else {
            //delayed left/top calculation when width/height are not set instantly
            setTimeout(setPosition, 300);
          }
        }
      };

      /**
       * onAdd is called when the map's panes are ready and the overlay has been
       * added to the map.
       */
      Overlay.prototype.onAdd = function() {
        var div = document.createElement('div');
        div.style.borderStyle = 'none';
        div.style.borderWidth = '0px';
        div.style.position = 'absolute';
        div.style.display = 'inline-block';
        div.style.zIndex = 10000;
        this._div = div;
        //div.style.visibility = "hidden";
        this.visible = true;

        var panes = this.getPanes();
        panes.overlayLayer.appendChild(div);
        panes.overlayMouseTarget.appendChild(div);

        self.clickListener = google.maps.event.addDomListener(div, 'click', function() {
          if (self.onClick !== undefined) {
            self.onClick(self.marker);
          }
        });
      };

      Overlay.prototype.draw = function() {
        if(!this._div) {
          return;
        }
        var div = this._div;
        div.innerHTML = self.$el.innerHTML;
        // We use the south-west and north-east
        // coordinates of the overlay to peg it to the correct position and size.
        // To do this, we need to retrieve the projection from the overlay.
        var overlayProjection = this.getProjection();


        /*
        // Retrieve the south-west and north-east coordinates of this overlay
        // in LatLngs and convert them to pixel coordinates.
        // We'll use these coordinates to resize the div.
        var sw = overlayProjection.fromLatLngToDivPixel(self.bounds.getSouthWest());
        var ne = overlayProjection.fromLatLngToDivPixel(self.bounds.getNorthEast());
        // Resize the image's div to fit the indicated dimensions.
        div.style.left = sw.x + 'px';
        div.style.top = ne.y + 'px';
        div.style.width = (ne.x - sw.x) + 'px';
        div.style.height = (sw.y - ne.y) + 'px';
        //console.log(div.style.left, div.style.top);
        */
        this.setPosition(self.position);
      };

      // The onRemove() method will be called automatically from the API if
      // we ever set the overlay's map property to 'null'.
      Overlay.prototype.onRemove = function() {
        this.map_ = undefined;
        this._div.remove();
        this._div = undefined;
      };

      /*
      Overlay.prototype.onRemove = function() {
        this.map_ = undefined;
      };
    */
      this.$overlay = new Overlay(this.$map);
    }

  }
};
</script>