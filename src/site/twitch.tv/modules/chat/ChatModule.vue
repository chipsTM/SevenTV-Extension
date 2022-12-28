<template>
	<template v-for="(inst, i) of chatList.instances" :key="inst.identifier">
		<ChatController v-if="dependenciesMet && isHookable" :list="inst" :controller="chatController.instances[i]" />
	</template>
</template>

<script setup lang="ts">
import { getTrackedNode, useComponentHook } from "@/common/ReactHooks";
import { useModule } from "@/composable/useModule";
import { computed } from "vue";
import ChatController from "./ChatController.vue";

const { dependenciesMet, markAsReady } = useModule("chat", {
	name: "Chat",
	depends_on: [],
});

const chatList = useComponentHook<Twitch.ChatListComponent>(
	{
		parentSelector: ".chat-list--default",
		predicate: (n) => n.scrollRef,
	},
	{
		trackRoot: true,
		hooks: {
			render: function (inst) {
				const nodes = inst.component.props.children.map((vnode) =>
					vnode.key ? getTrackedNode(inst, vnode.key as string, vnode) : null,
				);

				return nodes;
			},
		},
	},
);

const chatController = useComponentHook<Twitch.ChatControllerComponent>({
	parentSelector: ".chat-shell, .stream-chat",
	predicate: (n) => n.pushMessage && n.props?.messageHandlerAPI,
});

const isHookable = computed(() => chatController.instances.length === chatList.instances.length);

markAsReady();
</script>