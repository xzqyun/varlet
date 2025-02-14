<template>
  <div :class="n()">
    <slot />
  </div>
</template>

<script lang="ts">
import { defineComponent, computed, type ComputedRef } from 'vue'
import { props } from './props'
import { useFormItems, type FormProvider } from './provide'
import { createNamespace } from '../utils/components'
import { find } from '@varlet/shared'
import { getParentScroller, getTop, scrollTo, toPxNum } from '../utils/elements'
import { linear } from '../utils/shared'

const { n } = createNamespace('form')

export default defineComponent({
  name: 'VarForm',
  props,
  setup(props) {
    const disabled: ComputedRef<boolean> = computed(() => props.disabled)
    const readonly: ComputedRef<boolean> = computed(() => props.readonly)
    const { formItems, bindFormItems } = useFormItems()

    const scroll = (errorIndex: number, formItemElement: HTMLElement) => {
      if (errorIndex === -1 || !props.scrollToError) {
        return
      }

      // wait form-details animation end
      setTimeout(() => {
        const scroller = getParentScroller(formItemElement)
        const scrollerTop = scroller === window ? 0 : getTop(scroller as HTMLElement)
        const top = getTop(formItemElement) - scrollerTop - toPxNum(props.scrollToErrorOffsetY)
        scrollTo(scroller, {
          top,
          animation: linear,
        })
      }, 300)
    }

    // expose
    const validate = async () => {
      const res = await Promise.all(formItems.map(({ validate }) => validate()))

      const [error, errorIndex] = find(res, (r) => r === false, props.scrollToError)
      const formItemElement = formItems[errorIndex].instance.proxy?.$el

      scroll(errorIndex, formItemElement)

      return !!error
    }

    // expose
    const reset = () => formItems.forEach(({ reset }) => reset())

    // expose
    const resetValidation = () => formItems.forEach(({ resetValidation }) => resetValidation())

    const formProvider: FormProvider = {
      disabled,
      readonly,
    }

    bindFormItems(formProvider)

    return {
      n,
      validate,
      reset,
      resetValidation,
    }
  },
})
</script>

<style lang="less">
@import '../styles/common';
</style>
