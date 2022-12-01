<template>
  <div id="all">
    <h1>Dijkstra</h1>
    <div id="container"></div>
  </div>
</template>

<script>
import * as Three from "three";
export default {
  name: "Dijkstra",
  data() {
    return {
      scene: undefined,
      camera: undefined,
      renderer: undefined,
    };
  },
  methods: {
    init: function () {
      let container = document.getElementById("container");
      let tailleTab = 16;
      this.camera = new Three.PerspectiveCamera(
        70,
        container.clientWidth / container.clientHeight,
        0.01,
        20
      );
      this.camera.position.z = 15;
      this.camera.position.x = tailleTab / 2;
      this.camera.position.y = tailleTab / 2;

      this.scene = new Three.Scene();
      let tab = [];
      let tile = [];

      const geometry = new Three.PlaneGeometry(1, 1);
      const terre = new Three.MeshBasicMaterial({
        color: 0x32cd32,
        side: Three.DoubleSide,
      });
      const lave = new Three.MeshBasicMaterial({
        color: 0xed0000,
        side: Three.DoubleSide,
      });
      //generation map
      for (let i = 0; i < tailleTab; i++) {
        tab[i] = [];
        tile[i] = [];
        for (let j = 0; j < tailleTab; j++) {
          let mat = terre;
          tab[i][j] = "terre";
          if (Math.floor(Math.random() * 3) == 0) {
            mat = lave;
            tab[i][j] = "lave";
          }
          tile[i][j] = new Three.Mesh(geometry, mat);
          this.scene.add(tile[i][j]);
          //console.log(i + "  " +j)
          tile[i][j].position.x = i;
          tile[i][j].position.y = j;
        }
      }
      // generation clé
      let xkey = Math.floor(Math.random() * tailleTab);
      let ykey = Math.floor(Math.random() * tailleTab);
      tab[xkey][ykey] = "key";
      tile[xkey][ykey].material = new Three.MeshBasicMaterial({
        color: 0xffff00,
        side: Three.DoubleSide,
      });
      //generation hero
      let x = xkey;
      let y = ykey;
      while (x == xkey && y == ykey) {
        x = Math.floor(Math.random() * tailleTab);
        y = Math.floor(Math.random() * tailleTab);
      }
      tab[x][y] = "Hero";
      tile[x][y].material = new Three.MeshBasicMaterial({
        color: 0xf4fefe,
        side: Three.DoubleSide,
      });
      //------------------------------------------------------------------------------------------------------------------
      function isNewDirt(tab, x, y, dirts) {
        if (x >= 0 && x < tab.length && y >= 0 && y < tab[0].length) {
          return (
            tab[x][y] == "terre" && !dirts.find((d) => d[0] == x && d[1] == y)
          );
        }
      }

      function isKey(tab, x, y) {
        if (x >= 0 && x < tab.length && y >= 0 && y < tab[0].length) {
          return tab[x][y] == "key";
        }
      }

      function getKeyInfos(tab, startX, startY) {
        let dirts = [[startX, startY, 0]];
        let newDirtFound = true;
        let keyAccessible = false;
        let step = 0;
        let keyX = null;
        let keyY = null;

        while (newDirtFound && !keyAccessible) {
          step++;
          newDirtFound = false;
          const dirtsLength = dirts.length;
          for (let d = 0; d < dirtsLength; d++) {
            [
              { x: dirts[d][0], y: dirts[d][1] - 1 },
              { x: dirts[d][0], y: dirts[d][1] + 1 },
              { x: dirts[d][0] - 1, y: dirts[d][1] },
              { x: dirts[d][0] + 1, y: dirts[d][1] },
            ].forEach((pos) => {
              // Check new dirts
              if (isNewDirt(tab, pos.x, pos.y, dirts)) {
                dirts.push([pos.x, pos.y, step]);
                newDirtFound = true;
              }
              // Check key found
              if (isKey(tab, pos.x, pos.y)) {
                dirts.push([pos.x, pos.y, step]);
                keyAccessible = true;
                keyX = pos.x;
                keyY = pos.y;
              }
            });

            // Break for if key found
            if (keyAccessible) break;
          }
        }
        return {
          workTab: dirts,
          accessible: keyAccessible,
          pos: { x: keyX, y: keyY },
        };
      }

      function getPathToGo(tab, startX, startY, endX, endY) {
        let path = [[endX, endY]];

        for (let p = 0; p < path.length; p++) {
          let t = tab.find((c) => c[0] == path[p][0] && c[1] == path[p][1] + 1);
          let b = tab.find((c) => c[0] == path[p][0] && c[1] == path[p][1] - 1);
          let l = tab.find((c) => c[0] == path[p][0] + 1 && c[1] == path[p][1]);
          let r = tab.find((c) => c[0] == path[p][0] - 1 && c[1] == path[p][1]);

          let borders = [];
          t ? borders.push(t) : null;
          b ? borders.push(b) : null;
          l ? borders.push(l) : null;
          r ? borders.push(r) : null;

          let closer = borders.reduce(function (prev, curr) {
            return prev[2] < curr[2] ? prev : curr;
          });
          path.push(closer);
          if (closer[0] == startX && closer[1] == startY) break;
        }

        return path;
      }

      console.log("---START---");
      let heroX = x;
      let heroY = y;

      const key = getKeyInfos(tab, heroX, heroY);
      if (key.accessible) {
        const path = getPathToGo(
          key.workTab,
          heroX,
          heroY,
          key.pos.x,
          key.pos.y
        );
        highlightCheckCase(this.scene, path, 0xff00ff);
      }
      console.log("---END---");
      //------------------------------------------------------------------------------------------------------------------
      function highlightCheckCase(scene, tiletab, colortile) {
        for (let i = 0; i < tiletab.length; i++) {
          const geometry = new Three.PlaneGeometry(0.7, 0.7);
          const mat = new Three.MeshBasicMaterial({
            color: colortile,
            side: Three.DoubleSide,
          });
          const plane = new Three.Mesh(geometry, mat);
          scene.add(plane);
          plane.position.x = tiletab[i][0];
          plane.position.y = tiletab[i][1];
        }
      }

      this.renderer = new Three.WebGLRenderer({ antialias: true });
      this.renderer.setSize(container.clientWidth, container.clientHeight);
      container.appendChild(this.renderer.domElement);
    },
    animate: function () {
      requestAnimationFrame(this.animate);
      //this.mesh.rotation.x += 0.01;
      //this.mesh.rotation.y += 0.02;
      this.renderer.render(this.scene, this.camera);
    },
  },
  mounted() {
    this.init();
    this.animate();
  },
};
</script>

<style scoped>
#container {
  width: 1000px;
  height: 700px;
  margin-left: auto;
  margin-right: auto;
}
</style>
