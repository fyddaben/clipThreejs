<template>
  <div id="app">
    <div class="container J_container">
      <div id="viewport"></div>
      <div class="canvas-wrap">
        <canvas width='800' height='800' id='J_canvas'></canvas>
      </div>
    </div>
	  <div class="btn">开始绘画</div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      // 3D 世界是否可以拖动旋转
      isCanRotate: true,
      viewWidth: 0,
      viewHeight: 0,
      // 3D渲染
      render: {},
      // 场景
      scene: {},
      // 相机
      camera: {},
      // canvas 点
      canvasPoint: [],
      // 控制器
      orbitControls: {},
      // 2D画布是否可以画
      isDrawing: false,
    }
  },
  mounted() {

  },
  methods: {
    
    // 初始化3D场景
    init3D() {
      var container = document.getElementById('viewport')
			this.viewWidth = container.clientWidth
			this.viewHeight = container.clientHeight

			this.renderer = new THREE.WebGLRenderer({antialias: true});
			this.renderer.setSize( this.viewWidth, this.viewHeight);
      container.appendChild(this.renderer.domElement);
      
      this.scene = new THREE.Scene();

			this.scene.add(new THREE.AmbientLight(0x404040));
			var light = new THREE.DirectionalLight( 0xffffff );
			light.position.set( 1, 1, 15 ).normalize();
			this.scene.add( light );

			this.camera = new THREE.PerspectiveCamera(
				35,
				this.viewWidth / this.viewHeight,
				1,
				1000
			);
			this.camera.position.set( 0, 0, 15 );
			this.camera.lookAt( this.scene.position );
			this.scene.add( this.camera );
			this.wireMaterial = new THREE.MeshBasicMaterial({wireframe: true});
			// 增加控制器
			this.orbitControls = new THREE.OrbitControls(this.camera, this.renderer.domElement);
			this.orbitControls.enableDamping = true;
    }, 
    // 初始化二维画布
    init2D() {
      // 获取画布已经绘图上下文
			var canvas = document.getElementById("J_canvas");
			canvas.width = this.viewWidth
			canvas.height = this.viewHeight
			var context = canvas.getContext("2d");
			// 设置线条颜色
			context.strokeStyle = 'red';
			context.lineWidth = 1;
			context.lineJoin = 'round';
			context.lineCap = 'round';
			context.fillStyle= "rgba(100, 40, 40, 0.3)";
      this.canvas = canvas
      this.context = context
			// 画布添加鼠标事件
			canvas.addEventListener('mousedown', this.startDrawing, false);
			canvas.addEventListener('mousemove', this.draw, false);
			canvas.addEventListener('mouseup', this.stopDrawing, false);
    },
    startDrawing() {
      this.isDrawing = true
      var { x, y } = this.getPos(evt)
      
      this.canvasPoint.push([x, y])
    },
    getPos(evt){
      var top = document.getElementById('J_canvas').offsetTop
      var left = document.getElementById('J_canvas').offsetLeft
      return {
        x: evt.clientX - left,
        y: evt.clientY - top
      }
    }
  }
}
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
  html, body {
		margin: 0;
		padding: 0;
		overflow: hidden;
	}
	#viewport{
		width: 800px;
		height: 800px;
	}
	.btn{
		position: absolute;
		right: 100px;
		top: 50px;
		width: 100px;
		height: 40px;
		background: red;
		color: #fff;
		text-align: center;
		line-height: 40px;
		cursor: pointer;
	}
	.container{
		position: relative;
	}
	.canvas-wrap{
		position: absolute;
		width: 1px;
		height: 1px;
		left: 0;
		top: 0;
		overflow: hidden;
	}
	.container.show-canvas .canvas-wrap{
		width: 800px;
		height: 800px;
	}
</style>
