<template>
    <li class="c-tree__item-h">
        <div class="c-tree__item"
            :class="{ 'is-alias': isAlias }">
            <view-control class="c-tree__item__view-control"
                          :enabled="hasChildren"
                          v-model="expanded">
            </view-control>
            <object-label :domainObject="node.object"
                          :objectPath="node.objectPath">
            </object-label>
        </div>
        <ul v-if="expanded" class="c-tree">
            <tree-item v-for="child in children"
                       :key="child.id"
                       :node="child"
                       >
            </tree-item>
        </ul>
    </li>
</template>

<script>
    import viewControl from '../components/viewControl.vue';
    import ObjectLabel from '../components/ObjectLabel.vue';

    export default {
        name: 'tree-item',
        inject: ['openmct'],
        props: {
            node: Object
        },
        data() {
            return {
                hasChildren: false,
                isLoading: false,
                loaded: false,
                children: [],
                expanded: false
            }
        },
        computed: {
            isAlias() {
                let parent = this.node.objectPath[1];
                if (!parent) {
                    return false;
                }
                let parentKeyString = this.openmct.objects.makeKeyString(parent.identifier);
                return parentKeyString !== this.node.object.location;
            }
        },
        mounted() {
            // TODO: should update on mutation.
            // TODO: click navigation should not fubar hash quite so much.
            // TODO: should highlight if navigated to.
            // TODO: should have context menu.
            // TODO: should support drag/drop composition
            // TODO: set isAlias per tree-item

            this.domainObject = this.node.object;
            let removeListener = this.openmct.objects.observe(this.domainObject, '*', (newObject) => {
                this.domainObject = newObject;
            });
            this.$once('hook:destroyed', removeListener);
            if (this.openmct.composition.get(this.node.object)) {
                this.hasChildren = true;
            }
        },
        destroy() {
            if (this.composition) {
                this.composition.off('add', this.addChild);
                this.composition.off('remove', this.removeChild);
                delete this.composition;
            }
        },
        watch: {
            expanded(isExpanded) {
                if (!this.hasChildren) {
                    return;
                }
                if (!this.loaded && !this.isLoading) {
                    this.composition = this.openmct.composition.get(this.domainObject);
                    this.composition.on('add', this.addChild);
                    this.composition.on('remove', this.removeChild);
                    this.composition.load().then(this.finishLoading());
                }
            }
        },
        methods: {
            addChild (child) {
                this.children.push({
                    id: this.openmct.objects.makeKeyString(child.identifier),
                    object: child,
                    objectPath: [child].concat(this.node.objectPath)
                });
            },
            removeChild(identifier) {
                let removeId = this.openmct.objects.makeKeyString(identifier);
                this.children = this.children
                    .filter(c => c.id !== removeId);
            },
            finishLoading () {
                this.isLoading = false;
                this.loaded = true;
            }
        },
        components: {
            viewControl,
            ObjectLabel
        }
    }
</script>