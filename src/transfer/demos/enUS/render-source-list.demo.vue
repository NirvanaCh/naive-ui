<markdown>
# Custom source list
</markdown>

<script lang="ts">
import type { TransferRenderSourceList, TreeOption } from 'naive-ui'
import { NTree } from 'naive-ui'
import { repeat } from 'seemly'
import { defineComponent, h, ref } from 'vue'

function createLabel(level: number): string {
  if (level === 4)
    return 'Foo'
  if (level === 3)
    return 'Bar'
  if (level === 2)
    return 'Baz'
  if (level === 1)
    return '???'
  return ''
}

interface Option {
  label: string
  value: string
  children?: Option[]
}

function createData(level = 4, baseKey = ''): Option[] | undefined {
  if (!level)
    return undefined
  return repeat(6 - level, undefined).map((_, index) => {
    const value = `${baseKey}${level}${index}`
    return {
      label: createLabel(level),
      value,
      children: createData(level - 1, value)
    }
  })
}

function flattenTree(list: undefined | Option[]): Option[] {
  const result: Option[] = []
  function flatten(_list: Option[] = []) {
    _list.forEach((item) => {
      result.push(item)
      flatten(item.children)
    })
  }
  flatten(list)
  return result
}

export default defineComponent({
  setup() {
    const treeData = createData()
    const valueRef = ref<Array<string | number>>([])
    const renderSourceList: TransferRenderSourceList = function ({
      onCheck,
      pattern
    }) {
      return h(NTree, {
        style: 'margin: 0 4px;',
        keyField: 'value',
        checkable: true,
        selectable: false,
        blockLine: true,
        checkOnClick: true,
        data: treeData as unknown as TreeOption[],
        pattern,
        checkedKeys: valueRef.value,
        onUpdateCheckedKeys: (checkedKeys: Array<string | number>) => {
          onCheck(checkedKeys)
        }
      })
    }
    return {
      options: flattenTree(createData()),
      value: valueRef,
      renderSourceList
    }
  }
})
</script>

<template>
  <n-transfer
    v-model:value="value"
    :options="options"
    :render-source-list="renderSourceList"
    source-filterable
  />
</template>
