<template>
	<form
		class="properties"
		@submit.prevent="cancelSubmit"
		:key="node.id"
		v-if="node != null"
	>
		<accordion header="Base Properties" v-if="isInstanceNode">
			<texture-channel :node="getNode" :editor="editor" />
			<random-seed :node="getNode" />
		</accordion>
		<accordion header="Properties">
			<component
				v-for="(p, index) in this.properties"
				:is="p.componentName"
				:prop="p.prop"
				:propHolder="node"
				:editor="editor"
				@propertyChanged="propertyChanged"
				@property-change-completed="propertyChangeCompleted"
				:key="index"
			></component>
		</accordion>
	</form>
</template>

<script lang="ts">
import { Vue, Model, Prop, Component } from "vue-property-decorator";
import FloatPropertyView from "@/components/properties/FloatProp.vue";
import BoolPropertyView from "@/components/properties/BoolProp.vue";
import EnumPropertyView from "@/components/properties/EnumProp.vue";
import ColorPropertyView from "@/components/properties/ColorProp.vue";
import TextureChannelPropertyView from "@/components/properties/TextureChannelProp.vue";
import RandomSeedPropertyView from "@/components/properties/RandomSeedProp.vue";
import Accordion from "@/components/Accordion.vue";
import { Editor } from "@/lib/editor";
import { DesignerNode } from "@/lib/designer/designernode";
import { Property, IPropertyHolder } from "@/lib/designer/properties";
import GradientPropertyView from "@/components/properties/GradientProp.vue";
import StringPropertyView from "@/components/properties/StringProp.vue";
import ImagePropertyView from "@/components/properties/ImageProp.vue";
import {
	IProperyUi,
	PropertyChangeComplete
} from "../components/properties/ipropertyui";
import { UndoStack } from "@/lib/undostack";
import { PropertyChangeAction } from "@/lib/actions/propertychangeaction";

class PropHolder {
	prop: Property;
	componentName: string;
}

@Component({
	components: {
		floatView: FloatPropertyView,
		intView: FloatPropertyView,
		boolView: BoolPropertyView,
		enumView: EnumPropertyView,
		colorView: ColorPropertyView,
		gradientView: GradientPropertyView,
		stringView: StringPropertyView,
		imageView: ImagePropertyView,

		textureChannel: TextureChannelPropertyView,
		randomSeed: RandomSeedPropertyView,
		Accordion
	}
})
export default class NodePropertiesView extends Vue implements IProperyUi {
	@Prop()
	node: IPropertyHolder;

	@Prop()
	editor: Editor;

	propertyChanged(propName: string) {
		// if (this.editor.onnodepropertychanged)
		//   this.editor.onnodepropertychanged(self.node, prop);
	}

	// this is needed for undo-redo
	// some actions dont count as a full change until
	// the mouse button is up or the widget loses focus
	// like moving a slider
	// or typing text
	propertyChangeCompleted(evt: PropertyChangeComplete) {

		// let action = new PropertyChangeAction(
		// 	this,
		// 	evt.propName,
		// 	this.node,
		// 	evt.oldValue,
		// 	evt.newValue
		// );
		// UndoStack.current.push(action);
	}

	cancelSubmit() {
		return false;
	}

	// calculated
	get properties(): PropHolder[] {
		let props: PropHolder[] = this.node.properties.map(prop => {
			//let name: string = "";
			console.log(prop.type);

			// note: gotta transform nodes because some names like "image" are
			// built-in and arent compatible with the :is directive
			return {
				prop: prop,
				componentName: prop.type+"View"
			};
		});

		//console.log(props);
		return props;
	}

	get isInstanceNode() {
		return this.node instanceof DesignerNode;
	}

	get getNode(): DesignerNode {
		return this.node as DesignerNode;
	}

	refresh() {
		this.$forceUpdate();
	}
}
</script>

<style scoped>
.properties {
	background: #333;
}
</style>
