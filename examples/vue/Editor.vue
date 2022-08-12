<template>
  <div ref="editor"></div>
</template>

<script>
import { Editor, rootCtx, createCmdKey, commandsCtx, CommandsReady } from "@milkdown/core";
import { nord } from "@milkdown/theme-nord";
import { commonmark } from "@milkdown/preset-commonmark";
import { menu } from "@milkdown/plugin-menu";

export default {
  data() {
    return {
      editor: null,
    };
  },
  methods: {
    initEditorWithCustomCmdKey() {
      const CustomInsertImage = createCmdKey();
      const uploadImgCmd = () => async ctx => {
        // wait for command manager ready
        await ctx.wait(CommandsReady);
        const commandManager = ctx.get(commandsCtx);
        commandManager.create(CustomInsertImage, () => this.uploadImageHandler);
      };

      Editor.make()
        .config(ctx => {
          ctx.set(rootCtx, this.$refs.editor);
        })
        .use(nord)
        .use(commonmark)
        .use(uploadImgCmd)
        .use(
          menu({
            config: [
              // 其他菜单配置省略，实际需要写上！
              [
                {
                  type: "button",
                  icon: "image",
                  key: this.CustomInsertImage, // 使用自定义Key
                },
                // ...
              ],
            ],
          })
        )
        .create()
        .then(editor => (this.editor = editor));
    },
    uploadImageHandler() {
      // 本地选择图片
      let input = document.createElement("input");
      input.type = "file";
      input.click();
      input.addEventListener("change", e => {
        this.readAsDataURL(e.target.files[0]).then(url => {
          const view = this.editor.action(ctx => ctx.get(editorViewCtx));

          let tr = view.state.tr;
          if (!tr.selection.empty) tr.deleteSelection();

          // 插入至当前位置
          this.editor.action(ctx => {
            const view = ctx.get(editorViewCtx);
            const parser = ctx.get(parserCtx);
            const doc = parser(`![image](${url})`);
            if (!doc) return;
            const state = view.state;
            view.dispatch(
              state.tr.replace(tr.selection.from, tr.selection.from, new Slice(doc.content, 0, 0))
            );
          });
        });
      });
    },
    readAsDataURL(file) {
      var reader = new FileReader();
      return new Promise(function (accept, fail) {
        reader.onload = function () {
          return accept(reader.result);
        };
        reader.onerror = function () {
          return fail(reader.error);
        };
        return reader.readAsDataURL(file);
      });
    },
  },
};
</script>
