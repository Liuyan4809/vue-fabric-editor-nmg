<!--
 * @Author: 秦少卫
 * @Date: 2022-09-03 19:16:55
 * @LastEditors: 秦少卫
 * @LastEditTime: 2023-04-27 23:07:22
 * @Description: 导入模板
-->

<template>
  <div style="display: inline-block">
    <Divider plain orientation="left">{{ $t('title_template') }}</Divider>
    <Tooltip
      :content="item.label"
      v-for="(item, i) in state.list"
      :key="`${i}-bai1-button`"
      placement="top"
    >
      <img
        class="tmpl-img"
        :alt="item.label"
        v-lazy="item.src"
        @click="getTempData(item.tempUrl)"
      />
    </Tooltip>
  </div>
</template>

<script setup name="ImportTmpl">
import useSelect from '@/hooks/select';
import { downFontByJSON } from '@/utils/utils';
// import axios from 'axios';
import { Spin, Message } from 'view-ui-plus';
import { useI18n } from 'vue-i18n';
import aJSON from '@/template/a/a.json';
import aImg from '@/template/a/a.png';
import bJSON from '@/template/b/b.json';
import bImg from '@/template/b/b.png';

// const repoSrc = import.meta.env.APP_REPO;
const { t } = useI18n();
const { canvas } = useSelect();
// 加载模板数据
const state = reactive({
  jsonFile: null,
  list: [
    {
      label: '海报模板',
      tempUrl: aJSON,
      src: aImg,
    },
    {
      label: '旅游海报',
      tempUrl: bJSON,
      src: bImg,
    },
  ],
});
console.log('state', state.list);
// 插入文件
const insertSvgFile = () => {
  Spin.show({
    render: (h) => h('div', t('alert.loading_fonts')),
  });
  // 下载字体
  downFontByJSON(state.jsonFile)
    .then(() => {
      Spin.hide();
      // 使用指定的JSON数据填充画布
      canvas.c.loadFromJSON(state.jsonFile, () => {
        // 加载完毕后重新渲染
        canvas.c.renderAll.bind(canvas.c);
        setTimeout(() => {
          // 重新获取新的工作空间
          const workspace = canvas.c.getObjects().find((item) => item.id === 'workspace');
          // 设置工作空间不可编辑
          workspace.set('selectable', false);
          workspace.set('hasControls', false);
          // 将 renderAll 请求附加到下一个动画帧（异步）
          canvas.c.requestRenderAll();
          // 重新设置工作空间大小
          canvas.editor.editorWorkspace.setSize(workspace.width, workspace.height);
          // 渲染顶层canvas和辅助容器画布（同步）
          canvas.c.renderAll();
          canvas.c.requestRenderAll();
        }, 100);
      });
    })
    .catch(() => {
      Spin.hide();
      Message.error(t('alert.loading_fonts_failed'));
    });
};

// 获取模板列表数据
// const getTempList = () => {
//   Spin.show({
//     render: (h) => h('div', t('alert.loading_data')),
//   });
//   const getTemp = axios.get(`${repoSrc}template/index.json`);
//   getTemp
//     .then((res) => {
//       state.list = res.data.data.map((item) => {
//         item.tempUrl = repoSrc + item.tempUrl;
//         item.src = repoSrc + item.src;
//         return item;
//       });
//       Spin.hide();
//     })
//     .catch(Spin.hide);
// };

// 设置模板数据
const getTempData = (tmplUrl) => {
  Spin.show({
    render: (h) => h('div', t('alert.loading_data')),
  });
  state.jsonFile = JSON.stringify(tmplUrl);
  insertSvgFile();
  // const getTemp = axios.get(local + tmplUrl);
  // const getTemp = import('@/template/' + tmplUrl);
  // getTemp.then((res) => {
  //   state.jsonFile = JSON.stringify(res.data);
  //   insertSvgFile();
  // });
};

// getTempList();
</script>

<style scoped lang="less">
.tmpl-img {
  width: 94px;
  cursor: pointer;
  margin-right: 5px;
}
</style>
