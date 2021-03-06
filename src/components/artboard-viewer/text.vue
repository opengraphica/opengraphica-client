<template>
    <span></span>
</template>

<script>
import {
    DoubleSide, Mesh, Vector3, MeshBasicMaterial, ShapeBufferGeometry
} from 'three';
import { viewerPidToComponentMap } from '@/lib/viewer';

export default {
    name: 'ViewerText',
    props: {
        definition: {
            type: Object,
            default: () => ({})
        },
        pid: {
            type: String,
            default: null
        },
        scene: {
            type: Object,
            default: null
        }
    },
    watch: {
        'definition.position.x'(x) {
            this.setX(x);
        },
        'definition.position.y'(y) {
            this.setY(y);
        },
        'definition.rotation.x'(x) {
            this.setRotationX(x);
        },
        'definition.rotation.y'(y) {
            this.setRotationY(y);
        },
        'definition.rotation.z'(z) {
            this.setRotationZ(z);
        },
        'definition.scale.x'(x) {
            this.setScaleX(x);
        },
        'definition.scale.y'(y) {
            this.setScaleY(y);
        },
        'definition.scale.z'(z) {
            this.setScaleZ(z);
        },
        'pid'(pid) {
            viewerPidToComponentMap.delete(pid);
            viewerPidToComponentMap.set(pid, this);
        }
    },
    font: undefined,
    mesh: undefined,
    meshMaterial: undefined,
    selectionMesh: undefined,
    boundingBox: { min: { x: 0, y: 0 }, max: { x: 0, y: 0 } },
    created() {
        viewerPidToComponentMap.set(this.pid, this);
        this.meshMaterial = new MeshBasicMaterial({
            color: 0x006699,
            side: DoubleSide
        });
        this.font = this.$store.getters.resourceById(this.definition.fontResourceId).data;
        this.setText('Hello!');
        this.setX(this.definition.position.x);
        this.setY(this.definition.position.y);
        this.setZ(this.definition.position.z);
        this.setScaleX(this.definition.scale.x);
        this.setScaleY(this.definition.scale.y);
        this.setScaleZ(this.definition.scale.z);
        this.setRotationX(this.definition.rotation.x);
        this.setRotationY(this.definition.rotation.y);
        this.setRotationZ(this.definition.rotation.z);
        this.mesh.pid = this.pid;
        this.scene.add(this.mesh);
    },
    destroyed() {
        viewerPidToComponentMap.delete(this.pid);
        if (this.selectionMesh) {
            this.scene.remove(this.selectionMesh);
        }
        this.selectionMesh = undefined;
        this.scene.remove(this.mesh);
        this.mesh = undefined;
    },
    methods: {
        addSelection() {},
        getWidth(definition) {
            definition = definition || this.definition;
            if (this.definition.isAutoWidth) {
                return (this.boundingBox.max.x - this.boundingBox.min.x);
            } else {
                return definition.dimensions.w;
            }
        },
        getScaledWidth(definition) {
            definition = definition || this.definition;
            return this.getWidth(definition) * definition.scale.x;
        },
        getHeight(definition) {
            definition = definition || this.definition;
            if (this.definition.isAutoHeight) {
                return (this.boundingBox.max.y - this.boundingBox.min.y);
            } else {
                return definition.dimensions.h;
            }
        },
        getScaledHeight(definition) {
            definition = definition || this.definition;
            return this.getHeight(definition) * definition.scale.y;
        },
        getPosition(definition) {
            definition = definition || this.definition;
            this.mesh.parent.updateMatrixWorld();
            const worldPosition = this.mesh.getWorldPosition(new Vector3());
            return {
                x: worldPosition.x - Math.round(this.getScaledWidth(definition) / 2),
                y: worldPosition.y - Math.round(this.getScaledHeight(definition) / 2),
                z: worldPosition.z
            };
        },
        setText(text, definition) {
            const shapes = this.font.generateShapes(text, 32);
            const geometry = new ShapeBufferGeometry(shapes);
            geometry.computeBoundingBox();
            this.boundingBox = geometry.boundingBox;
            const centerX = -0.5 * (geometry.boundingBox.max.x - geometry.boundingBox.min.x);
            const centerY = -0.5 * (geometry.boundingBox.max.y - geometry.boundingBox.min.y);
            geometry.translate(centerX, centerY, 0);
            this.mesh = new Mesh(geometry, this.meshMaterial);
        },
        setX(x, definition) {
            definition = definition || this.definition;
            this.mesh.position.x = Math.round(x + (this.getScaledWidth(definition) / 2));
            if (this.selectionMesh) {
                this.selectionMesh.position.x = this.mesh.position.x;
            }
            this.$root.$emit('artboard::' + this.pid.split('.')[0] + '::draw');
        },
        setY(y, definition) {
            definition = definition || this.definition;
            this.mesh.position.y = Math.round(y + (this.getScaledHeight(definition) / 2));
            if (this.selectionMesh) {
                this.selectionMesh.position.y = this.mesh.position.y;
            }
            this.$root.$emit('artboard::' + this.pid.split('.')[0] + '::draw');
        },
        setZ(z, definition) {
            definition = definition || this.definition;
            this.mesh.position.z = z;
            if (this.selectionMesh) {
                this.selectionMesh.position.z = this.mesh.position.z + 0.001;
            }
            this.$root.$emit('artboard::' + this.pid.split('.')[0] + '::draw');
        },
        setRotationX(x, definition) {
            definition = definition || this.definition;
            x = (x + 180) * Math.PI/180;
            this.mesh.rotation.x = x;
            if (this.selectionMesh) {
                this.selectionMesh.rotation.x = x;
            }
        },
        setRotationY(y, definition) {
            definition = definition || this.definition;
            y = y * Math.PI/180;
            this.mesh.rotation.y = y;
            if (this.selectionMesh) {
                this.selectionMesh.rotation.y = y;
            }
        },
        setRotationZ(z, definition) {
            definition = definition || this.definition;
            z = z * Math.PI/180;
            this.mesh.rotation.z = z;
            if (this.selectionMesh) {
                this.selectionMesh.rotation.z = z;
            }
        },
        setScaleX(x, definition) {
            definition = definition || this.definition;
            this.mesh.scale.x = x;
            if (this.selectionMesh) {
                this.selectionMesh.scale.x = x;
            }
            this.setX(definition.position.x, definition);
        },
        setScaleY(y, definition) {
            definition = definition || this.definition;
            this.mesh.scale.y = y;
            if (this.selectionMesh) {
                this.selectionMesh.scale.y = y;
            }
            this.setY(definition.position.y, definition);
        },
        setScaleZ(z, definition) {
            definition = definition || this.definition;
            this.mesh.scale.z = z;
            if (this.selectionMesh) {
                this.selectionMesh.scale.z = z;
            }
        },
        removeSelection() {
            if (this.selectionMesh) {
                this.scene.remove(this.selectionMesh);
                this.selectionMesh = undefined;
            }
        }
    }
};
</script>