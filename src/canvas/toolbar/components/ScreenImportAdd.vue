
<template>
     <div class="MatcToolbarItem  MatcMultiIcon MatcScreenImportAdd MatcToolbarDropDownButton">
							<div type="button" data-dojo-attach-point="button">
								<label data-dojo-attach-point="label" class="">
									<span data-dojo-attach-point="icon" class=""></span>
									<span class="mdi mdi-plus-circle MatcTinyIcon"></span>
								</label>

							 </div>
							 <div class="MatcToolbarPopUp" role="menu" data-dojo-attach-point="popup">
							 	<div class="MatcScreenAddCntr MatcPadding">
									 <div class="container-fluid">
										<div class="row">
											<div class="col-md-4 MatcCenter" data-dojo-attach-point="addCntr">
											</div>
											<div class="col-md-4 MatcCenter" data-dojo-attach-point="addRotate">
											</div>
											<div class="col-md-4 MatcCenter" data-dojo-attach-point="addFB">
											</div>
											<div class="col-md-4 MatcCenter" data-dojo-attach-point="addYT">
											</div>
											<div class="col-md-4 MatcCenter" data-dojo-attach-point="addIG">
											</div>
											<div class="col-md-4 MatcCenter" data-dojo-attach-point="uploadCntr">
												<!-- <span class="MatcUploaderIcon MatcMiddle mdi mdi-cloud-upload"></span> -->
											</div>
										</div>
									</div>
								</div>

							  	<div class="MatcToolbarPopUpArrowCntr">
								  	<div class="MatcToolbarPopUpArrow">
								  	</div>
							  	</div>
							  </div>
						  </div>
</template>
<script>
import DojoWidget from 'dojo/DojoWidget'
import css from 'dojo/css'
import lang from 'dojo/_base/lang'
import on from 'dojo/on'
import touch from 'dojo/touch'
import topic from 'dojo/topic'
import DomBuilder from 'common/DomBuilder'
import Util from 'core/Util'
import RenderFactory from 'core/RenderFactory'
import _DropDown from './_DropDown'

export default {
    name: 'ScreenImportAdd',
    mixins:[Util, DojoWidget, _DropDown],
    data: function () {
        return {
            screenWidth: 300,
            screenHeight: 600,
            selectedCategory: "Screen",
            showSubCatgeoryLabels: false,
						previewSizes : {
							"default" : {
								w : 120,
								h : 70
							},
							"Screen" : {
								w : 160,
								h : 200
							}
						}
        }
    },
    components: {},
    methods: {
			setModel (m){
				this.model = m;
				this.screenWidth = m.screenSize.w;
				this.screenHeight = m.screenSize.h;
				this.renderFactory = new RenderFactory();
				this.renderFactory.setModel(m);
				css.add(this.icon, this.getAppTypeIcon(m));

			},

			onVisible (){
				css.add(this.domNode,"MatcToolbarItemActive");
				if(this.uploader){
					this.uploader.initFileDnD(this.popup);
				}
				this.setInfo("( Click or Drop files here)");
			},

			onHide (){
				if(this.domNode){
					css.remove(this.domNode,"MatcToolbarItemActive");
					if (this.uploader) {
						this.uploader.destroyFileDnD();
					}
				}
			},

			init (){

				var db = new DomBuilder();

				var add = db.div("MatcUploader").build(this.addCntr);
				db.div("MatcUploaderIcon MatcMiddle mdi mdi-crop-portrait").build(add);
				db.div("MatcHint MatcMarginTop", "Create Empty Screen").build(this.addCntr);
				this.own(on(add, touch.press, lang.hitch(this, "onAddScreen", false, 'none')));

				var addRotate = db.div("MatcUploader").build(this.addRotate);
				// db.div("MatcUploaderIcon MatcMiddle mdi mdi-crop-portrait").build(addRotate);
				db.div("MatcHint MatcMarginTop", "Create Rotated Screen").build(this.addRotate);
				this.own(on(addRotate, touch.press, lang.hitch(this, "onAddScreen", true, 'rotate')));

				var addFB = db.div("MatcUploader").build(this.addFB);
				db.div("MatcHint MatcMarginTop", "Create Facebook Cover").build(this.addFB);
				this.own(on(addFB, touch.press, lang.hitch(this, "onAddScreen", false, 'fb')));

				var addYT = db.div("MatcUploader").build(this.addYT);
				db.div("MatcHint MatcMarginTop", "Create Youtube Banner").build(this.addYT);
				this.own(on(addYT, touch.press, lang.hitch(this, "onAddScreen", false, 'yt')));

				var addIG = db.div("MatcUploader").build(this.addIG);
				db.div("MatcHint MatcMarginTop", "Create Instagram Post").build(this.addIG);
				this.own(on(addIG, touch.press, lang.hitch(this, "onAddScreen", false, 'ig')));

				var upload = db.div("MatcUploader").build(this.uploadCntr);
				db.div("MatcUploaderIcon MatcMiddle mdi mdi-cloud-upload").build(upload);
				db.div("MatcHint MatcMarginTop", "Import Screens").build(this.uploadCntr);
				this.own(on(upload, touch.press, lang.hitch(this, "onImportScreen")));

			},

			onImportScreen (e) {
				this.stopEvent(e);
				this.hideDropDown();
				this.emit("onImport", e);
			},


			onAddScreen (rotate, type, e){
				this.stopEvent(e);
				var screen = this.createEmptyScreen(0,0,"Screen", rotate, type);
				screen._type = "Screen";
				this.hideDropDown();
				this.emit("onAdd", screen, e);
			},


			onFilesSelected (files,e){
				this.logger.log(0,"onFilesSelected", "enter" );
				topic.publish("matc/canvas/startupload", files, e);
				this.hideDropDown();
			},



			onFilesUploaded (result,e ){
				this.logger.log(1, "ScreenImportAdd.onFilesUploaded", "enter");
				var uploads  = result.uploads;
				var screens = [];
				for(var i=0; i< uploads.length; i++){
					var upload = uploads[i];
					var screen = this.createEmptyScreen((this.model.screenSize.w + 100 )*i, 0, upload.name);
					screen.style.backgroundImage ={
						url : upload.url,
						w : upload.width,
						h : upload.height
					};
					screens.push(screen);
				}
				this.hideDropDown();
				this.emit("onUpload", screens,e);

			},

			setInfo (txt){
				if(this.help){
					css.remove(this.help, "MatcError");
					this.help.innerHTML=txt;
				}

			},

			onError (txt){
				if(this.help){
					css.add(this.help, "MatcError");
					this.help.innerHTML=txt;
				}
			},

			destroy (){
				if(this.uploader){
					this.uploader.destroy();
				}
			}
    },
    mounted () {
    }
}
</script>