<!--
 * @Author: 秦少卫
 * @Date: 2022-09-03 19:16:55
 * @LastEditors: 秦少卫
 * @LastEditTime: 2023-05-07 09:34:18
 * @LastEditors: 秦少卫
 * @LastEditTime: 2023-04-10 14:33:18

 * @Description: 保存文件
-->

<template>
  <div class="save-box">
    <Button style="margin-left: 10px" type="text" @click="beforeClear">
      {{ $t('empty') }}
    </Button>
    <Dropdown style="margin-left: 10px" @on-click="saveWith">
      <Button type="primary">
        {{ $t('keep') }}
        <Icon type="ios-arrow-down"></Icon>
      </Button>
      <template #list>
        <DropdownMenu>
          <DropdownItem name="clipboard">{{ $t('copy_to_clipboard') }}</DropdownItem>
          <DropdownItem name="saveImg">{{ $t('save_as_picture') }}</DropdownItem>
          <DropdownItem name="saveSvg">{{ $t('save_as_svg') }}</DropdownItem>
          <DropdownItem name="saveJson" divided>{{ $t('save_as_json') }}</DropdownItem>
          <DropdownItem name="uploadToTemplateLibrary" divided>上传至模板库</DropdownItem>
        </DropdownMenu>
      </template>
    </Dropdown>
    <Modal v-model="uploadToTemplateLibraryVisible" title="上传至模板库" width="60%" @on-ok="onClickOk">
      <Form :model="currentForm.tableData">
        <Table :columns="columns" :data="currentForm.tableData">
          <template #id="{ row }">
            <img :src="row._element.currentSrc" alt="" style="max-width: 100px; max-height: 100px" />
          </template>
          <template #replace="{ row, index }">
            <FormItem>
              <Select v-model="currentForm.tableData[index].replaceType" :max-tag-count="2" transfer clearable>
                <Option v-for="item in replaceTypeOptions" :value="item.value" :key="item.value">
                {{ item.label }}
                </Option>
              </Select>
            </FormItem>
          </template>
        </Table>
      </Form>
    </Modal>
  </div>
</template>

<script setup name="save-bar">
import { Modal } from 'view-ui-plus';
import { clipboardText } from '@/utils/utils.ts';
import useSelect from '@/hooks/select';
import { v4 as uuid } from 'uuid';
import { debounce } from 'lodash-es';
import { useI18n } from 'vue-i18n';
const { t } = useI18n();

const { canvas } = useSelect();

const currentForm = ref({
  tableData: [],
});

const columns = ref([
  {
    title: 'id',
    key: 'id',
    slot: 'id',
  },
  {
    title: 'name',
    key: 'name',
  },
  {
    title: '类型',
    key: 'type',
  },
  {
    title: '替换成分',
    key: 'replace',
    slot: 'replace',
    minWidth: 200,
  },
]);

const replaceTypeOptions = ref([
  {
    label: '作为标题元素',
    value: 'title'
  },
  {
    label: '作为描述替换元素',
    value: 'describe'
  },
  {
    label: '作为图片替换元素',
    value: 'image'
  },
  {
    label: '作为logo替换元素',
    value: 'logo'
  },
])

// 上传至模板库模态框是否可见
let uploadToTemplateLibraryVisible = ref(false);

const cbMap = {
  clipboard() {
    const jsonStr = canvas.editor.getJson();
    clipboardText(JSON.stringify(jsonStr, null, '\t'));
  },

  saveJson() {
    const dataUrl = canvas.editor.getJson();
    const fileStr = `data:text/json;charset=utf-8,${encodeURIComponent(
      JSON.stringify(dataUrl, null, '\t')
    )}`;
    downFile(fileStr, 'json');
  },

  saveSvg() {
    const workspace = canvas.c.getObjects().find((item) => item.id === 'workspace');
    const { left, top, width, height } = workspace;
    const dataUrl = canvas.c.toSVG({
      width,
      height,
      viewBox: {
        x: left,
        y: top,
        width,
        height,
      },
    });
    const fileStr = `data:image/svg+xml;charset=utf-8,${encodeURIComponent(dataUrl)}`;
    downFile(fileStr, 'svg');
  },

  saveImg() {
    const workspace = canvas.c.getObjects().find((item) => item.id === 'workspace');
    canvas.editor.ruler.hideGuideline();
    const { left, top, width, height } = workspace;
    const option = {
      name: 'New Image',
      format: 'png',
      quality: 1,
      left,
      top,
      width,
      height,
    };
    canvas.c.setViewportTransform([1, 0, 0, 1, 0, 0]);
    const dataUrl = canvas.c.toDataURL(option);
    downFile(dataUrl, 'png');
    canvas.editor.ruler.showGuideline();
  },

  uploadToTemplateLibrary() {
    uploadToTemplateLibraryVisible.value = true;
    const objs = canvas.c.getObjects().filter((item) => 'image' === item.type);
    currentForm.value.tableData = [
      ...objs.map(({ id, name, type, _element, replaceType }) => ({ id, name, type, _element, replaceType })),
    ];
    const dataUrl = canvas.editor.getJson();
    // JSON.stringify(dataUrl, null, '\t')
    console.log('uploadToTemplateLibrary', dataUrl);
  },
};

const saveWith = debounce(function (type) {
  cbMap[type] && typeof cbMap[type] === 'function' && cbMap[type]();
}, 300);

/**
 * @desc clear canvas 清空画布
 */
const clear = () => {
  canvas.c.getObjects().forEach((obj) => {
    if (obj.id !== 'workspace') {
      canvas.c.remove(obj);
    }
  });
  canvas.c.discardActiveObject();
  canvas.c.renderAll();
};

const onClickOk = () => {

  let allObjs = canvas.c.getObjects();
  let tableData = currentForm.value.tableData

  // 设置替换类型
  for (let i = 0, len = allObjs.length; i < len; i++) {
    let current = allObjs[i];
    let replaceItem = tableData.find(item => item.id === current.id)
    current.set({
      replaceType: replaceItem ? replaceItem.replaceType : null
    })
  }

  const dataUrl = canvas.editor.getJson();
  // JSON.stringify(dataUrl, null, '\t')
  console.log('onClickOk', dataUrl);
  const fileStr = `data:text/json;charset=utf-8,${encodeURIComponent(
    JSON.stringify(dataUrl, null, '\t')
  )}`;

  cbMap.saveImg()

  setTimeout(function() {
    downFile(fileStr, 'json');
  }, 1000)

}

const beforeClear = () => {
  Modal.confirm({
    title: t('tip'),
    content: `<p>${t('clearTip')}</p>`,
    okText: t('ok'),
    cancelText: t('cancel'),
    onOk: () => clear(),
  });
};

function downFile(fileStr, fileType) {
  const anchorEl = document.createElement('a');
  anchorEl.href = fileStr;
  anchorEl.download = `${uuid()}.${fileType}`;
  document.body.appendChild(anchorEl); // required for firefox
  anchorEl.click();
  anchorEl.remove();
}
</script>

<style scoped lang="less">
.save-box {
  display: inline-block;
  padding-right: 10px;
}
</style>
