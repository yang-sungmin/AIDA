<template lang="html">
    <v-list-tile id='tool-tile'>
        <v-btn @click.native="initialiseTool" flat block id='tool'>
            <v-icon :class="{'grey--text text--darken-2': !this.active,
                             'blue--text text--darken-1': this.active}">
                    crop_landscape
            </v-icon>
        </v-btn>
    </v-list-tile>

</template>

<script>
import paper from 'paper';

import { mapActions } from 'vuex';
import { mapState } from 'vuex';
import { mapGetters } from 'vuex';


export default {
    props: ['active'],

    data() {
        return {
            toolRect: null,
            strokeWidth: 400, // Default value, will be updated relative to view
        }
    },

    computed: {
        ...mapState({
            paperScope: state => state.annotation.paperScope,
            viewportZoom: state => state.image.viewer.viewport.getZoom(true),
            imageWidth: state => state.image.viewer.world.getItemAt(0).getContentSize().x
        })
    },

    methods: {
        ...mapActions([
            'prepareCanvas'
        ]),

        ...mapGetters([
            'getDefaultColor',
        ]),

        initialiseTool() {

            // Prepare PaperJS canvas for interaction.
            this.prepareCanvas();

            // Activate the paperJS tool.
            this.toolRect.activate();

            // Set the default strokewidth relative to image size and zoom.
            this.strokeWidth = this.imageWidth/(this.viewportZoom*500);
        }
    },

    created() {

        // On drag draw feedforward shadow rectangle in realtime.
        const toolDrag = (event) => {

        	let trackingRect = new paper.Path.Rectangle(event.downPoint, event.point);
                trackingRect.strokeColor = new paper.Color({hue: 220, saturation: 0.7, lightness: 0.5, alpha: 1});
                trackingRect.strokeWidth = this.strokeWidth;

        	// Constantly update tracking rect by removing it and re-drawing.
            trackingRect.removeOn({
                drag: true,
                up:true
            });
        };

        // Finalise rectangle properties and draw.
        const toolUp = (event) => {

        	let newRect = new paper.Path.Rectangle(event.downPoint, event.point);
            newRect.strokeColor = new paper.Color(this.getDefaultColor().stroke);
            newRect.fillColor = new paper.Color(this.getDefaultColor().fill);
            newRect.strokeWidth = this.strokeWidth;
        };

        this.toolRect = new paper.Tool();
        this.toolRect.onMouseDrag = toolDrag;
        this.toolRect.onMouseUp = toolUp;
    }
}
</script>

<style lang="css">
#tool {
    min-width: 0px;
}

#tool-tile {
    padding: 0px;
}
</style>
