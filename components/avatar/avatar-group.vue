<template>
  <div :class="className" role="group" :style="style">
    <div v-for="(option, index) in renderAvatars" :key="index" :class="nh.be('item')">
      <slot :option="option" :index="index">
        <Avatar
          inherit
          :src="option.src"
          :icon="option.icon"
          :alt="option.alt"
          :fit="option.fit"
          :src-set="option.srcSet"
          :gap="option.gap"
          :icon-scale="option.iconScale"
          :fallback-src="option.fallbackSrc"
        >
          {{ option.text }}
        </Avatar>
      </slot>
    </div>
    <div v-if="restAvatars.length" :class="[nh.be('item'), nh.bem('item', 'rest')]">
      <Tooltip
        v-if="props.showTip"
        inherit
        :trigger="props.tipTrigger"
        :tip-class="nh.be('rest')"
      >
        <template #trigger>
          <slot name="rest" :options="restAvatars" :count="restAvatars.length">
            <Avatar inherit :color="props.restColor" :background="props.restBackground">
              {{ `+${restAvatars.length}` }}
            </Avatar>
          </slot>
        </template>
        <slot name="tip" :options="restAvatars" :count="restAvatars.length">
          <Avatar
            v-for="(option, index) in restAvatars"
            :key="index"
            inherit
            :src="option.src"
            :icon="option.icon"
            :alt="option.alt"
            :fit="option.fit"
            :src-set="option.srcSet"
            :gap="option.gap"
            :icon-scale="option.iconScale"
            :fallback-src="option.fallbackSrc"
          >
            {{ option.text }}
          </Avatar>
        </slot>
      </Tooltip>
      <slot
        v-else
        name="rest"
        :options="restAvatars"
        :count="restAvatars.length"
      >
        <Avatar inherit :color="props.restColor" :background="props.restBackground">
          {{ `+${restAvatars.length}` }}
        </Avatar>
      </slot>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watchEffect, provide } from 'vue'
import { Avatar } from '@/components/avatar'
import { Tooltip } from '@/components/tooltip'
import { useNameHelper, useProps } from '@vexip-ui/config'
import { avatarGroupProps } from './props'
import { GROUP_STATE } from './symbol'

import type { ComponentSize } from '@vexip-ui/config'
import type { AvatarOption } from './symbol'

export default defineComponent({
  name: 'AvatarGroup',
  components: {
    Avatar,
    Tooltip
  },
  props: avatarGroupProps,
  setup(_props) {
    const props = useProps('avatarGroup', _props, {
      size: 'default' as ComponentSize,
      options: {
        default: () => [],
        static: true
      },
      circle: false,
      max: null,
      showTip: false,
      tipTrigger: 'hover',
      vertical: false,
      offset: null,
      restColor: null,
      restBackground: null
    })

    const nh = useNameHelper('avatar-group')

    const renderAvatars = ref<AvatarOption[]>([])
    const restAvatars = ref<AvatarOption[]>([])

    watchEffect(() => {
      const size = props.options.length

      if (props.max > 0 && size > props.max) {
        renderAvatars.value = props.options.slice(0, props.max - 1)
        restAvatars.value = props.options.slice(props.max - 1)
      } else {
        renderAvatars.value = Array.from(props.options)
        restAvatars.value = []
      }
    })

    provide(GROUP_STATE, props)

    const className = computed(() => {
      return {
        [nh.b()]: true,
        [nh.ns('avatar-vars')]: true,
        [nh.bm('inherit')]: props.inherit,
        [nh.bm(props.size)]: typeof props.size !== 'number' && props.size !== 'default',
        [nh.bm('circle')]: props.circle,
        [nh.bm('vertical')]: props.vertical
      }
    })
    const style = computed(() => {
      const style: Record<string, string> = {}

      if (typeof props.offset === 'number') {
        style[nh.cv('offset')] = `${props.offset}px`
      }

      return style
    })

    return {
      props,
      nh,
      renderAvatars,
      restAvatars,

      className,
      style
    }
  }
})
</script>
