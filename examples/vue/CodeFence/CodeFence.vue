<script lang="ts">
import { defineComponent } from 'vue';
import { languageListSlice } from './CodeFence';
import { useNodeCtx } from '@milkdown/vue';
import { Node } from '@milkdown/prose/model';

export default defineComponent({
    name: 'CodeFence',
});
</script>

<script setup lang="ts">
import { ref } from 'vue';
const metadata = useNodeCtx<Node>();
const attrs = metadata.node.attrs;
const languages = metadata.ctx.get(languageListSlice) ?? [];

const filename = ref(attrs['filename'] ?? '');
const lang = attrs['language'] ?? '';

const showInput = ref(attrs['showInput'] ?? false);

const onChange = (e: Event) => {
    const view = metadata.view;
    const node = metadata.node;
    const getPos = metadata.getPos;
    const { target } = e;
    if (!(target instanceof HTMLSelectElement)) {
        return;
    }
    const { value } = target;
    if (!view.editable) {
        target.value = node.attrs['language'];
        return;
    }
    const { tr } = view.state;
    view.dispatch(
        tr.setNodeMarkup(getPos(), undefined, {
            ...node.attrs,
            language: value,
        }),
    );
};

const toggleInput = () => {
    const view = metadata.view;
    const node = metadata.node;
    const getPos = metadata.getPos;
    if (!view.editable) {
        return;
    }
    const show = node.attrs['showInput'];
    const { tr } = view.state;
    view.dispatch(
        tr.setNodeMarkup(getPos(), undefined, {
            ...node.attrs,
            showInput: !show,
        }),
    );
    showInput.value = !show;
};

const onKeydown = (e: KeyboardEvent) => {
    e.stopPropagation();
};

const onFilenameChange = (e: Event) => {
    const view = metadata.view;
    const node = metadata.node;
    const getPos = metadata.getPos;

    const { target } = e;
    if (!(target instanceof HTMLInputElement)) return;
    if (!view.editable) {
        return;
    }
    const { value } = target;

    const { tr } = view.state;
    view.dispatch(
        tr.setNodeMarkup(getPos(), undefined, {
            ...node.attrs,
            filename: value,
            showInput: false,
        }),
    );
    showInput.value = false;
    filename.value = value;
};
</script>

<template>
    <div class="code-fence">
        <div class="control">
            <span class="filename">
                {{ filename || 'No File Name' }}
                <input v-if="showInput" @keydown="onKeydown" @change="onFilenameChange" />
                <button v-else @click="toggleInput">edit</button>
            </span>
            <select @change="onChange">
                <option v-for="language in languages" :selected="language === lang" :value="language">
                    {{ language }}
                </option>
            </select>
        </div>
        <div class="code">
            <slot />
        </div>
    </div>
</template>

<style scoped>
.code-fence {
    border: 1px solid #ccc;
    padding: 10px;
}

.control {
    margin-bottom: 10px;
    display: flex;
    justify-content: space-between;
}

.code {
    background: #ccc;
    margin: 0 auto !important;
    white-space: pre;
    border-radius: 4px;
    padding: 10px;
}
</style>
