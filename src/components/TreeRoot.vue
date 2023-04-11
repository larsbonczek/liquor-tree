<template>
  <component
    :is="tag"
    role="tree"
    :class="{'liquor-tree': true, 'liquor-tree-loading': this.loading, 'tree--draggable' : !!this.draggableNode}"
  >
    <template v-if="filter && matches.length == 0">
      <div
        class="liquor-tree-filter-empty"
        v-html="opts.filter.emptyText"
      />
    </template>
    <template v-else>
      <ul
        class="liquor-tree-root"
        @dragstart="onDragStart"
      >
        <template v-if="opts.filter.plainList && matches.length > 0">
          <TreeNode
            v-for="node in visibleMatches"
            :key="node.id"
            :node="node"
            :options="opts"
          />
        </template>
        <template v-else>
          <TreeNode
            v-for="node in visibleModel"
            :key="node.id"
            :node="node"
            :options="opts"
          />
        </template>
      </ul>
    </template>

    <DraggableNode
      v-if="draggableNode"
      :target="draggableNode"
    />
  </component>
</template>

<script>
  import TreeNode from './TreeNode.vue'
  import DraggableNode from './DraggableNode.vue'
  import TreeMixin from '../mixins/TreeMixin.js'
  import TreeDnd from '../mixins/DndMixin.js'
  import Tree from '../lib/Tree.js'

  const defaults = {
    direction: 'ltr',
    multiple: true,
    checkbox: false,
    checkOnSelect: false,
    autoCheckChildren: true,
    autoDisableChildren: true,
    checkDisabledChildren: true,
    parentSelect: false,
    keyboardNavigation: true,
    nodeIndent: 24,
    minFetchDelay: 0,
    fetchData: null,
    propertyNames: null,
    deletion: false,
    dnd: false,
    editing: false,
    onFetchError: function(err) { throw err }
  }

  const filterDefaults = {
    emptyText: 'Nothing found!',
    matcher(query, node) {
      const isMatched = new RegExp(query, 'i').test(node.text)

      if (isMatched) {
        if (node.parent && new RegExp(query, 'i').test(node.parent.text)) {
          return false
        }
      }

      return isMatched
    },
    plainList: false,
    showChildren: true
  };

  export default {
    name: 'Tree',
    components: {
      TreeNode,
      DraggableNode
    },

    mixins: [TreeMixin, TreeDnd],

    provide: _ => ({
      tree: null
    }),

    props: {
      data: {},

      options: {
        type: Object,
        default: _ => ({})
      },

      filter: String,

      tag: {
        type: String,
        default: 'div'
      }
    },

    data () {
      // we should not mutating a prop directly...
      // that's why we have to create a new object
      // TODO: add method for changing options
      let opts = Object.assign({}, defaults, this.options)

      opts.filter = Object.assign(
        {},
        filterDefaults,
        opts.filter
      )

      return {
        model: [],
        tree: null,
        loading: false,
        opts,
        matches: [],
        draggableNode: null
      }
    },

    computed: {
      visibleModel() {
        return this.model.filter(function(node) {
          return node && node.visible()
        })
      },
      visibleMatches() {
        return this.matches.filter(function(node) {
          return node && node.visible()
        })
      }
    },

    watch: {
      filter (term) {
        this.tree.filter(term)
      }
    },
  }
</script>

<style>
  .liquor-tree {
    overflow: auto;
  }

  .liquor-tree-root,
  .liquor-tree-children {
    list-style: none;
    padding: 0;
  }

  .liquor-tree > .liquor-tree-root,
  .liquor-tree > .liquor-tree-filter-empty {
    padding: 3px;
    box-sizing: border-box;
  }

  .liquor-tree.tree--draggable .liquor-tree-node:not(.selected) > .liquor-tree-content:hover {
    background: transparent;
  }

  .drag-above,
  .drag-below,
  .drag-on {
    position: relative;
    z-index: 1;
  }

  .drag-on > .liquor-tree-content {
    background: #fafcff;
    outline: 1px solid #7baff2;
  }

  .drag-above > .liquor-tree-content::before, .drag-below > .liquor-tree-content::after {
    display: block;
    content: '';
    position: absolute;
    height: 8px;
    left: 0;
    right: 0;
    z-index: 2;
    box-sizing: border-box;
    background-color: #3367d6;
    border: 3px solid #3367d6;
    background-clip: padding-box;
    border-bottom-color: transparent;
    border-top-color: transparent;
    border-radius: 0;
  }

  .drag-above > .liquor-tree-content::before {
    top: 0;
    transform: translateY(-50%);
  }

  .drag-below > .liquor-tree-content::after {
    bottom: 0;
    transform: translateY(50%);
  }
</style>
