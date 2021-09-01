<template>
    <TroisCanvas
        :cameraPosition="[0, 0, 20]"
        :rendererProperties="rendererProperties"
    >
        <!-- lighting -->
        <ambientLight color="#808080" />
        <pointLight
            ref="light"
            :castShadow="true"
            :shadow-mapSize-height="1024"
            :shadow-mapSize-width="1024"
        />

        <!-- boxes -->
        <instancedMesh
            ref="imesh"
            :castShadow="true"
            :receiveShadow="true"
            :args="[null, null, NUM_INSTANCES]"
            @added="update"
        >
            <boxGeometry :args="[SIZE, SIZE, SIZE]" />
            <meshPhongMaterial color="#9C1E15" />
        </instancedMesh>

        <!-- plane -->
        <mesh
            :position-z="-10 - SIZE"
            @pointerOver="onPointerMove"
            :receiveShadow="true"
        >
            <planeGeometry :args="[W * 2, H * 2]" />
            <meshPhongMaterial color="#9C1E15" />
        </mesh>
    </TroisCanvas>
</template>

<script lang="ts">
// port of Trois Shadows demo:
// https://troisjs.github.io/examples/shadows.html
import { defineComponent } from 'vue'
import { Object3D, PCFSoftShadowMap, Vector2 } from 'three'

import SimplexNoise from 'simplex-noise'
const simplex = new SimplexNoise()

export default defineComponent({
    setup() {
        const SIZE = 1.5,
            NX = 20,
            NY = 20,
            PADDING = 1
        const SIZEP = SIZE + PADDING
        const W = NX * SIZEP - PADDING
        const H = NY * SIZEP - PADDING
        const blank = new Object3D()
        const mousePos = new Vector2()
        return {
            SIZE,
            NX,
            NY,
            PADDING,
            SIZEP,
            W,
            H,
            NUM_INSTANCES: NX * NY,
            blank,
            mousePos,
            imesh: null as THREE.InstancedMesh | null,
            rendererProperties: {
                'shadowMap.type': PCFSoftShadowMap,
                'shadowMap.enabled': true,
            },
        }
    },
    methods: {
        onPointerMove(evt: any) {
            this.mousePos = evt.intersection.point
            const light = (this.$refs.light as any)?.$el?.instance
            light.position.set(
                this.mousePos.x,
                this.mousePos.y,
                light.position.z
            )
        },
        update() {
            requestAnimationFrame(this.update)

            const pointer = this.mousePos

            const imesh = (this.$refs as any)?.imesh?.$el?.instance
            if (!imesh) return

            const x0 = -this.W / 2 + this.PADDING
            const y0 = -this.H / 2 + this.PADDING
            const time = Date.now() * 0.0001
            const mx = pointer.x * 0.01
            const my = pointer.y * 0.01
            const noise = 0.005
            let x, y, nx, ny, rx, ry
            for (let i = 0; i < this.NX; i++) {
                for (let j = 0; j < this.NY; j++) {
                    x = x0 + i * this.SIZEP
                    y = y0 + j * this.SIZEP
                    nx = x * noise + mx
                    ny = y * noise + my
                    rx = simplex.noise3D(nx, ny, time) * Math.PI
                    ry = simplex.noise3D(ny, nx, time) * Math.PI
                    this.blank.position.set(x, y, -10)
                    this.blank.rotation.set(rx, ry, 0)
                    this.blank.updateMatrix()
                    imesh.setMatrixAt(i * this.NY + j, this.blank.matrix)
                }
            }
            imesh.instanceMatrix.needsUpdate = true
        },
    },
})
</script>