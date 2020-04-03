<template>
  <div id="app">
    <div class="container J_container" v-bind:class="{'show-canvas':isShowCanvas}">
      <div id="viewport"></div>
      <div class="canvas-wrap">
        <canvas width="800" height="800" id="J_canvas"></canvas>
      </div>
    </div>
    <div class="btn" @click='startCanvas'>开始绘画</div>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  name: "App",
  data() {
    return {
      // 3D 世界是否可以拖动旋转
      isCanRotate: true,
      viewWidth: 0,
      viewHeight: 0,
      isShowCanvas: false,
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
      // 初始化立方体
      cube_mesh: {},
    };
  },
  mounted() {
    this.init3D()
    this.initBox()
    this.init2D()
    this.renderView()
  },
  methods: {
    // 初始化3D场景
    init3D() {
      var container = document.getElementById("viewport");
      this.viewWidth = container.clientWidth;
      this.viewHeight = container.clientHeight;

      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(this.viewWidth, this.viewHeight);
      container.appendChild(this.renderer.domElement);

      this.scene = new THREE.Scene();

      this.scene.add(new THREE.AmbientLight(0x404040));
      var light = new THREE.DirectionalLight(0xffffff);
      light.position.set(1, 1, 15).normalize();
      
      this.scene.add(light)

      this.light = light

      this.camera = new THREE.PerspectiveCamera(
        35,
        this.viewWidth / this.viewHeight,
        1,
        1000
      );
      this.camera.position.set(0, 0, 15);
      this.camera.lookAt(this.scene.position);
      this.scene.add(this.camera);
      this.wireMaterial = new THREE.MeshBasicMaterial({ wireframe: true });
      // 增加控制器
      this.orbitControls = new THREE.OrbitControls(
        this.camera,
        this.renderer.domElement
      );
      this.orbitControls.enableDamping = true;
    },
    // 初始化一个立方体
    initBox() {
      
      var cube_geometry = new THREE.BoxBufferGeometry( 3, 3, 3 );
      const normalMaterial = new THREE.MeshNormalMaterial();
      var material = new THREE.MeshBasicMaterial( {color: 0x00ff00} );
      
      var cube_mesh = new THREE.Mesh( cube_geometry, this.wireMaterial );
      cube_mesh.position.x = 0;
      this.cube_mesh = cube_mesh
      this.scene.add( cube_mesh );
    },

    // 初始化二维画布
    init2D() {
      // 获取画布已经绘图上下文
      var canvas = document.getElementById("J_canvas");
      canvas.width = this.viewWidth;
      canvas.height = this.viewHeight;
      var context = canvas.getContext("2d");
      // 设置线条颜色
      context.strokeStyle = "red";
      context.lineWidth = 1;
      context.lineJoin = "round";
      context.lineCap = "round";
      context.fillStyle = "rgba(100, 40, 40, 0.3)";
      this.canvas = canvas;
      this.context = context;
      // 画布添加鼠标事件
      canvas.addEventListener("mousedown", this.startDrawing, false);
      canvas.addEventListener("mousemove", this.draw, false);
      canvas.addEventListener("mouseup", this.stopDrawing, false);
    },
    startCanvas() {
      this.isCanRotate = false
      this.isShowCanvas = true
    },
    startDrawing(evt) {
      this.isDrawing = true;
      var { x, y } = this.getPos(evt);

      this.canvasPoint.push([x, y]);
    },
    draw(evt) {
      if (!this.isDrawing) return;
      evt.preventDefault();
      evt.stopPropagation();
      const { x, y } = this.getPos(evt);
      this.canvasPoint.push([x, y]);
    },
    stopDrawing() {
      this.isDrawing = false
      this.isShowCanvas = false
      this.turnNum(this.canvasPoint)
    },
    // 获取near plane的宽高
    getNearPlaneWidth() {
      var cameraposition = this.camera.position
      var camera = this.camera
      var distance = Math.sqrt(Math.pow(cameraposition.x, 2) + Math.pow(cameraposition.y, 2) + Math.pow(cameraposition.z, 2))
      
      var vFOV = THREE.Math.degToRad( camera.fov ); // convert vertical fov to radians

      var dheight = 2 * Math.tan( vFOV / 2 ) * distance; // visible height

      var dwidth = dheight * camera.aspect; 
      return {
        w: dwidth,
        h: dheight
      }
    },
    // 把2D坐标，转化为3D nearPlane的坐标
    turnNum(point) {
      var newPoint = []
      var w = this.getNearPlaneWidth().w
      var h = this.getNearPlaneWidth().h
      var decPoint = []
      point.map((p, i)=> {
        var x = p[0] / this.viewWidth * 2 - 1;
        var y = -p[1] / this.viewHeight * 2 + 1;
        newPoint.push([x*(w/2), y*(h/2)])
        decPoint.push([x, y])
      })
      this.isCanRotate = true
      this.drawShape(newPoint, decPoint)
    },
    // 计算点在轮廓中是否存在
    pointInpolygon(point, vs) {
      // ray-casting algorithm based on
      // http://www.ecse.rpi.edu/Homepages/wrf/Research/Short_Notes/pnpoly.html

      var x = point[0], y = point[1];

      var inside = false;
      for (var i = 0, j = vs.length - 1; i < vs.length; j = i++) {
        var xi = vs[i][0], yi = vs[i][1];
        var xj = vs[j][0], yj = vs[j][1];

        var intersect = ((yi > y) != (yj > y))
          && (x < (xj - xi) * (y - yi) / (yj - yi) + xi);
        if (intersect) inside = !inside;
      }
      return inside;
    },
    drawShape(point, decPoint) {
      var x = 0, y = 0;
      var heartShape = new THREE.Shape();
      point.map((p, i)=> {
        if (i == 0) {
          heartShape.moveTo(p[0], p[1])
        } else {
          heartShape.lineTo(p[0], p[1])
        }
      })
      var w = this.getNearPlaneWidth().w
			var h = this.getNearPlaneWidth().h
      // 计算平面的点与相机形成的射线与物体的交点
      var raycaster = new THREE.Raycaster();
      var allInterPoint = []
      // 交点的原值
      var allInterPointVal = []
      // 原shape的值
      var originshape = []
      var cameraposition = this.camera.position
      // decPoint 只是轮廓数据，需要得到平面的所有点
      var unit = 0.01 //横移单位
      var xArr = []
      var yArr = []
      decPoint.map((v, i) => {
        xArr.push(v[0])
        yArr.push(v[1])
      })
      var decMax_x = Math.max(...xArr)
      var decMin_x = Math.min(...xArr)
      var decMax_y = Math.max(...yArr)
      var decMin_y = Math.min(...yArr)

      // 按照单位增加点
      var decPointPlugin = []
      for (var i = decMin_x; i <= decMax_x; i = i + unit) {
        for (var b = decMin_y; b <= decMax_y; b = b + unit) {
          if (this.pointInpolygon([i, b], decPoint)) {
            decPointPlugin.push([i, b])
          }
        }
      }

      decPoint = decPoint.concat(decPointPlugin)
      decPoint.map((v, i) => {
        var mouse = new THREE.Vector2(v[0], v[1]);

        raycaster.setFromCamera(mouse, this.camera);
        var intersection = raycaster.intersectObject(this.cube_mesh);
        if (intersection.length > 0) {

          var interp = intersection[0].point
          allInterPoint.push(Math.sqrt(Math.pow(interp.x - cameraposition.x, 2) + Math.pow(interp.y - cameraposition.y, 2) + Math.pow(interp.z - cameraposition.z, 2)))
          allInterPointVal.push(interp)
          originshape.push([v[0] * (w / 2), v[1] * (h / 2)])
        }
      })

      var max = 0;

      for (var i = 1; i < allInterPoint.length; i++) {
        var cur = allInterPoint[i];
        cur < allInterPoint[max] ? max = i : 0
      }
      // 查询最小的值
      var target = allInterPointVal[max]

      // 获取这个target对应的shape的点
      var originshapepoint = originshape[max]
      // 因为根据相机，转了角度
      var angle = this.camera.rotation

      // 获取转向camera之后的点的坐标
      var a = new THREE.Euler( angle.x, angle.y, angle.z, 'XYZ' );
      var b = new THREE.Vector3( originshapepoint[0], originshapepoint[1], 0);
      b.applyEuler(a);
      //addPoint([b.x, b.y, b.z], 0xFFC0CB) // 原点
      // 要计算偏移量
      var offset = [target.x - b.x, target.y - b.y, target.z - b.z]

      // 计算被删减者的长度
				var depth = 10

      //console.log(depth, 'wwww')
      var extrudeSettings = {
          steps: 10,
          depth: -depth,
          bevelEnabled: false,
          bevelThickness: 1,
          bevelSize: 1,
          bevelSegments: 1
      };
      const normalMaterial = new THREE.MeshNormalMaterial();
      var Sgeometry = new THREE.ExtrudeBufferGeometry( heartShape, extrudeSettings );
      var focalPoint = new THREE.Vector3().copy(cameraposition)
      // 调整转向
      Sgeometry.lookAt(focalPoint);
      // 再移动到相机的点与当前点的中间
		  Sgeometry.translate(offset[0], offset[1], offset[2]);
      
      var Smesh = new THREE.Mesh( Sgeometry,  this.wireMaterial) ;
      
      // 进行切割
      const box3 = threecsg.intersect(this.cube_mesh, Smesh, normalMaterial);

      //const box3 = threecsg.subtract(boxMesh, Smesh, normalMaterial);
      //box3.quaternion.set(0, 0, 0, 1).normalize();
      this.scene.add( Smesh );
      this.scene.add( box3 );
    },
    getPos(evt) {
      var top = document.getElementById("J_canvas").offsetTop;
      var left = document.getElementById("J_canvas").offsetLeft;
      return {
        x: evt.clientX - left,
        y: evt.clientY - top
      };
    },

		drawLine() {
      this.context.beginPath()
      this.canvasPoint.map((v, b)=> {
        if (b == 0) {
          this.context.moveTo(v[0], v[1])
        } else {
          this.context.lineTo(v[0], v[1])
        }
      })
      this.context.lineTo(this.canvasPoint[0][0], this.canvasPoint[0][1])
      this.context.stroke()
      this.context.fill()
    },
    renderView() {
      if (this.isCanRotate) {
        this.orbitControls.update()
      }
      if (this.isDrawing) {
        this.context.clearRect(0, 0, this.canvas.width, this.canvas.height)
        this.drawLine()
      }
      requestAnimationFrame( this.renderView );
      this.light.position.copy(this.camera.position);
      
      this.renderer.render(this.scene, this.camera);
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
html,
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}
#viewport {
  width: 800px;
  height: 800px;
}
.btn {
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
.container {
  position: relative;
}
.canvas-wrap {
  position: absolute;
  width: 1px;
  height: 1px;
  left: 0;
  top: 0;
  overflow: hidden;
}
.container.show-canvas .canvas-wrap {
  width: 800px;
  height: 800px;
}
</style>
