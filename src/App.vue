<template>
  <Renderer
    ref="renderer"
    antialias
    orbit-ctrl
    resize
    shadow
    @click="randomColors"
  >
    <Camera :position="{ z: 3.5 }" />
    <Scene background="white">
      <AmbientLight color="#aaaaaa" />
      <SpotLight
        color="#ffffff"
        :intensity="2"
        :position="{ x: 0, y: 10, z: 8 }"
        :angle="Math.PI / 17"
        :penumbra="1"
        cast-shadow
        :shadow-map-size="{ width: 1024, height: 1024 }"
      />
      <Plane :width="5" :height="5" :position="{ z: -0.1 }" receive-shadow>
        <StandardMaterial color="#aaaaaa" :props="{ roughness: 0.2 }">
          <Texture
            :props="woodTextureProps"
            src="https://assets.codepen.io/33787/Wood048_1K_Color.jpg"
          />
          <Texture
            :props="woodTextureProps"
            src="https://assets.codepen.io/33787/Wood048_1K_NormalGL.jpg"
            name="normalMap"
          />
          <Texture
            :props="woodTextureProps"
            src="https://assets.codepen.io/33787/Wood048_1K_Roughness.jpg"
            name="roughnessMap"
          />
        </StandardMaterial>
      </Plane>
      <CannonWorld :gravity="{ x: 0, y: -4, z: 0 }" @before-step="onBeforeStep">
        <InstancedMesh
          ref="imeshC"
          :count="COUNT"
          @created="initIMesh"
          cast-shadow
        >
          <SphereGeometry
            :radius="0.1"
            :width-segments="16"
            :height-segments="16"
          />
          <PhysicalMaterial
            :props="{
              transparent: true,
              transmission: 1,
              thickness: 0.4,
              roughness: 0.1,
            }"
          />
        </InstancedMesh>
        <Box
          v-for="i in 3"
          :key="`box-${i}`"
          :width="3"
          :height="0.05"
          :depth="0.2"
          :position="{ x: (i % 2 === 0 ? 1 : -1) * 1, y: (i - 2) * 1.5, z: 0 }"
          :rotation="{ z: ((i % 2 === 0 ? 1 : -1) * Math.PI) / 6 }"
          cast-shadow
        >
          <PhongMaterial color="#aaaaaa">
            <Texture
              :props="woodTextureProps"
              src="https://assets.codepen.io/33787/Wood048_1K_Color.jpg"
            />
          </PhongMaterial>
        </Box>
      </CannonWorld>
    </Scene>
  </Renderer>
</template>

<script>
import { Color, MathUtils, Object3D, RepeatWrapping } from 'three';
import {
  AmbientLight,
  Box,
  Camera,
  InstancedMesh,
  PhongMaterial,
  Plane,
  Renderer,
  Scene,
  SphereGeometry,
  SpotLight,
  StandardMaterial,
  Texture,
  PhysicalMaterial,
} from 'troisjs';
import CannonWorld from '@troisjs/components/src/physics/CannonWorld.js';

import niceColors from 'nice-color-palettes';

const { randFloat: rnd, randFloatSpread: rndFS } = MathUtils;

export default {
  components: {
    AmbientLight,
    Box,
    Camera,
    InstancedMesh,
    PhongMaterial,
    Plane,
    Renderer,
    Scene,
    SphereGeometry,
    SpotLight,
    StandardMaterial,
    Texture,
    PhysicalMaterial,
    CannonWorld,
  },
  setup() {
    return {
      COUNT: 250,
      woodTextureProps: {
        repeat: { x: 3, y: 3 },
        wrapS: RepeatWrapping,
        wrapT: RepeatWrapping,
      },
    };
  },
  methods: {
    initIMesh(imesh) {
      this.imesh = imesh;

      const dummy = new Object3D();
      const color = new Color();
      // const cscale = chroma.scale(niceColors[Math.floor(rnd(0, 100))]);
      const scales = new Array(this.COUNT);
      const masses = new Array(this.COUNT);
      const colors = new Array(this.COUNT)
        .fill()
        .map(() => niceColors[95][Math.floor(rnd(0, 5))]);

      for (let i = 0; i < this.COUNT; i++) {
        dummy.position.set(rndFS(2), rndFS(5), 0);
        const scale = rnd(0.2, 1);
        scales[i] = scale;
        masses[i] = scale * 0.1;
        dummy.scale.set(scale, scale, scale);
        dummy.updateMatrix();
        imesh.setMatrixAt(i, dummy.matrix);
        imesh.setColorAt(i, color.set(colors[i]));
        // imesh.setColorAt(i, color.set(cscale(rnd(0, 1)).hex()));
      }
      imesh.instanceMatrix.needsUpdate = true;
      imesh.userData.scales = scales;
      imesh.userData.masses = masses;
      imesh.userData.damping = 0.7;
    },
    onBeforeStep() {
      this.imesh.userData.bodies.forEach((body) => {
        if (body.position.y < -3) {
          body.position.set(rndFS(2), rnd(3, 5), 0);
          body.angularVelocity.set(0, 0, 0, 0);
          body.velocity.set(0, 0, 0);
          body.quaternion.set(0, 0, 0, 1);
        }
      });
    },
    randomColors() {
      const ci = Math.floor(rnd(0, 100));
      const color = new Color();
      const colors = new Array(this.COUNT)
        .fill()
        .map(() => niceColors[ci][Math.floor(rnd(0, 5))]);
      for (let i = 0; i < this.COUNT; i++) {
        this.imesh.setColorAt(i, color.set(colors[i]));
      }
      this.imesh.instanceColor.needsUpdate = true;
    },
  },
};
</script>

<style>
body,
html,
#app {
  margin: 0;
  height: 100%;
}
canvas {
  display: block;
}
</style>
