<template>
  <el-form ref="form" :model="field" label-width="80px">
    <template v-for="item in f_items">
      <el-form-item
        :key="item.prop"
        :prop="item.prop"
        :label="item.label"
        :required="item.required"
        :rules="item.rules"
      >
        <slot
          v-if="item.type === 'slot'"
          :name="item.slot_name"
          :data="item"
        ></slot>
        <component
          v-else
          :value.sync="field[item.prop]"
          :config="item"
          :is="
            !item.type
              ? 'e-text'
              : item.type === 'text'
              ? 'e-text'
              : `e-${item.type}`
          "
        />
      </el-form-item>
    </template>

    <el-form-item style="text-align: right; margin-top: 50px">
      <el-button
        v-for="(button, i) in buttons"
        :key="button.key"
        :type="button.type || 'primary'"
        @click="handlerButton({ button, i })"
        >{{ button.label }}</el-button
      >
    </el-form-item>
  </el-form>
</template>
<script>
import createRules from "./createRules";

/**
 * require.context(dir,sub,extension)
 * @description: 自动化装配组件
 * @param {String} directory 读取目录
 * @param {Boolean} subdirectories 是否读取目录下的子文件
 * @param {String} extension 读取的文件，可选正则表达
 * @param {String} alias 读取文件别名
 */
const files = require.context("@/components/common/control",true,/\index.vue$/);
const [path, modules, alias] = [require("path"), {}, "e-"];
files.keys().forEach((key) => (modules[`${alias}${key.split("/")[1]}`] = files(key).default));

export default {
  name: "e-form",
  components: modules,
  props: {
    items: {
      type: Array,
      default: [],
    },
    field: {
      type: Object,
      default: () => ({}),
    },
    buttons: {
      type: Array,
      default: () => [],
    },
    beforeSubmit: {
      type: Function,
    },
  },
  data() {
    return {
      f_items: [],
    };
  },
  computed: {
    cssStyle() {
      const [] = [];
      return {
        // al
      };
    },
  },
  beforeMount() {
    // this.f_items = createRules(this.items); // 官方不建议直接在html中使用props接收的属性
    this.f_items = this.items; // 官方不建议直接在html中使用props接收的属性
  },
  methods: {
    /**
     * 表单按钮事件
     */
    handlerButton(data) {
      const { button, i } = data;
      button.key === "confirm" && this.submit(data);
      button.key === "cancel" && this.cancel(data);
      //   其他按钮
      button.cb && button.cb(data);
    },
    // 提交
    submit(data) {
      this.$refs.form.validate((valid) => {
        if (valid) {
          if (typeof this.beforeSubmit === "function") {
            this.$set(data, "loading", true);
            this.beforeSubmit()
              .then(({ code, msg }) => {
                this.$set(data, "loading", false);
                this.$message({
                  type: code === 200 ? "success" : "error",
                  message: msg,
                });
              })
              .catch((err) => {
                this.$set(data, "loading", false);
              });
          }
        }
      });
    },
    // 取消
    cancel(data) {
      const { button, i } = data;
      this.$refs.form.resetFields();
      button.cb && button.cb(data);
    },
  },
};
</script>