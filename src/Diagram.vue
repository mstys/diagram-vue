<template>
  <div class="scrollXY">
    <div>
      <svg
        :width="width * scale"
        :height="height * scale"
        xmlns="http://www.w3.org/2000/svg"
      >
        <g :transform="scaleStr">
          <rect
            x="0"
            y="0"
            :width="width"
            :height="height"
            :fill="background"
            @click="reset"
          />
          <Link
            :width="width"
            :height="height"
            :link="item"
            v-for="item in linkList"
            :selected="item.id === selectedLink"
            :key="item.id"
            :source="findNode(item.source)"
            :destination="findNode(item.destination)"
            :editable="editable"
            :labels="labels"
            :scale="scale"
            @editLink="editLink"
            @select="selectLink"
            @updateLocation="updateLinkLocation"
            @remove="removeLink"
          />
          <Node
            :width="width"
            :height="height"
            :node="item"
            :selected="item.id === selectedNode"
            v-for="item in nodeList"
            :key="item.id"
            :createLinkMode="createLinkMode"
            :editable="editable"
            :labels="labels"
            :scale="scale"
            @editNode="editNode"
            @select="selectNode"
            @updateLocation="updateNodeLocation"
            @toggleSelect="toggleSrcSelect"
            @commitDest="commitDest"
            @remove="removeNode"
          />
        </g>
      </svg>
    </div>
  </div>
</template>
<script>
import Node from "@/lib/Node";
import Link from "@/lib/Link";
export default {
  name: "Diagram",
  props: {
    width: Number,
    height: Number,
    scale: {
      type: String,
      default: "1"
    },
    background: String,
    nodes: Array,
    links: Array,
    editable: Boolean,
    labels: Object
  },
  components: {
    Node,
    Link
  },
  computed: {
    scaleStr() {
      return (
        "scale(" + (this.scale || 1.0) + ")" + "translate(" + 0 + "," + 0 + ")"
      );
    },
    nodeList: {
      get() {
        return this.nodes;
      },
      set(val) {
        this.$emit("nodeChanged", {
          nodes: val
        });
      }
    },
    linkList: {
      get() {
        return this.links;
      },
      set(val) {
        this.$emit("linkChanged", {
          links: val
        });
      }
    }
  },
  data() {
    return {
      name: "",
      url: "",
      color: "",
      selectedNode: -1,
      selectedLink: -1,
      createLinkMode: false
    };
  },
  methods: {
    editNode(item) {
      this.$emit("editNode", item);
    },
    editLink(item) {
      this.$emit("editLink", item);
    },
    generateID() {
      return (
        new Date().getTime().toString(16) +
        Math.floor(Math.random() * 1000000).toString(16)
      );
    },
    addNode() {
      if (!this.editable) return;
      this.nodeList.push({
        id: this.generateID(),
        content: {
          text: this.name,
          color: this.color,
          url: this.url
        },
        width: 200,
        height: 60,
        point: {
          x: 10,
          y: 100 + Math.random() * 100
        }
      });
    },
    reset() {
      if (!this.createLinkMode) {
        this.selectedNode = -1;
        this.selectedLink = -1;
      }
    },
    updateLinkLocation(obj) {
      let item = this.linkList.find(x => x.id === obj.id);
      item.point.x = obj.x;
      item.point.y = obj.y;
    },
    findNode(id) {
      return this.nodes.find(x => x.id === id);
    },
    removeLink(id) {
      this.linkList = this.linkList.filter(x => x.id !== id);
    },
    updateNodeLocation(obj) {
      let item = this.nodeList.find(x => x.id === obj.id);
      item.point.x = obj.x;
      item.point.y = obj.y;
    },
    selectNode(id) {
      this.selectedNode = id;
    },
    selectLink(id) {
      this.selectedLink = id;
    },
    toggleSrcSelect() {
      this.createLinkMode = !this.createLinkMode;
    },
    commitDest(id) {
      let src = this.nodeList.find(x => x.id === this.selectedNode);
      let dest = this.nodeList.find(x => x.id === id);
      this.linkList.push({
        id: this.generateID(),
        source: this.selectedNode,
        destination: id,
        point: {
          x: (src.point.x + dest.point.x) * 0.5,
          y: (src.point.y + dest.point.y) * 0.5
        }
      });
      this.createLinkMode = false;
      this.selectedNode = -1;
    },
    removeNode(id) {
      const nodes = this.nodeList.filter(x => x.id !== id);
      this.nodeList = nodes;
      const links = this.linkList.filter(x => {
        return x.source !== id && x.destination !== id;
      });
      this.linkList = links;
    }
  }
};
</script>
<style>
.button {
  cursor: pointer;
}
.grab {
  cursor: grab;
}
</style>
