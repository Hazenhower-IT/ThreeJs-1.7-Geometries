<template>
  <canvas ref="canvasRef"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted} from "vue"
import { OrbitControls} from "three/examples/jsm/controls/OrbitControls"
import * as THREE from "three"

var controls


let canvasRef = ref();

//Utilizziamo i valori del viewport per impostare la grandezza del canvas
const sizes = {
  width: window.innerWidth,
  height: window.innerHeight
}

//Aggiungiamo l'event listner per il resize del canvas
window.addEventListener("resize", ()=>{
  //update sizes for the canvas
  sizes.width = window.innerWidth
  sizes.height = window.innerHeight

  //update the camera aspect ratio
  camera.aspect = sizes.width / sizes.height
  camera.updateProjectionMatrix()

  //update renderer
  renderer.setSize(sizes.width, sizes.height)
  //HANDLE PIXEL RATIO
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
})


//HANDLE FULL SCREEN (Il codice aggiuntivo è per permettere il funzionamento corretto anche con browser Safari)
window.addEventListener("dblclick",()=>{

  const fullscreenElement = document.fullscreenElement ||  document.webkitFullscreenElement
  if(!fullscreenElement){
    if(canvasRef.value.requestFullscreen){
      canvasRef.value.requestFullscreen()
    }
    else if(canvasRef.value.webkitRequestFullscreen){
      canvasRef.value.webkitRequestFullscreen()
    }
    
  }
  else{
    if(document.exitFullscreen)
    {
      document.exitFullscreen()
    }
    else if(document.webkitExitFullscreen){
      document.webkitExitFullscreen()
    }
  }
})

let scene = new THREE.Scene()

let renderer


//Built-in Geometries
//Tutte queste geometrie ereditano dalla classe geometry, e quindi posseggono tutte le classi ed i metodi di questa proprietà

//GEOMETRY CUSTOM
/* 
const geometry = new THREE.BufferGeometry()

const vertices = new Float32Array([
  0, 0, 0,
  0, 1, 0,
  1, 0, 0
])

geometry.setAttribute("position", new THREE.BufferAttribute(vertices, 3))

const customMesh = new THREE.Mesh(
  geometry,
  new THREE.MeshBasicMaterial({color:0x0000ff, wireframe:true})
)
//scene.add(customMesh)

*/

//RANDOM VERTICES GENERATOR
const geometry = new THREE.BufferGeometry();

const count = 50 //quanti triangoli vogliamo

const positionsArray = new Float32Array(count * 3 * 3) //50 triangoli, ogni triangolo composto da 3 vertici, ogni vertice composto da 3 punti

for(let i=0; i<count * 3 * 3; i++){
  positionsArray[i] = (Math.random() - 0.5) * 4
}

const positionsAttribute = new THREE.BufferAttribute(positionsArray, 3) //3 sono quanti parametri deve prendere per ciascun vertex
geometry.setAttribute("position",positionsAttribute)

//N.B. Alcune geometrie hanno facce in cui i triangolo condividono gli stessi vertici, e questo puo causare problemi
// per evitarli si assegnano degli indici ai vertex, in questo modo avremo meno dati da mandare alla gpu(meno vertici), ma è piu difficile per organizzare il codice

const customMesh = new THREE.Mesh(
  geometry,
  new THREE.MeshBasicMaterial({color:0x0000ff, wireframe:true})
)
scene.add(customMesh)

// BOX GEOMETRY
//la box geometry ha 6 parametri : width (larghezza), height (altezza), depth (profondità), widthSegments (quanti segmenti su asse x), heightSegments (quanti segmenti su asse y), depthSegments (quanti segmenti su asse z)
//questi segmenti corrispondo a quanti "triangoli" compongono la faccia dell'immagine
const boxGeometry = new THREE.BoxGeometry(1,1,1,2,2,2)
const boxMaterial = new THREE.MeshBasicMaterial({color: 0xff0000, wireframe: true}) 
const box = new THREE.Mesh(boxGeometry,boxMaterial)
box.position.set(0 , 0, 0)
//scene.add(box)

// PLANE GEOMETRY
// una classe che genere geometrie "piane" (adatta per creare il piano di base di un gioco)
const plane = new THREE.Mesh(
  new THREE.PlaneGeometry(25,25),
  new THREE.MeshBasicMaterial({color: 0x0000ff, side: THREE.DoubleSide})
)
//scene.add(plane)

//CIRCLE GEOMETRY
const cerchio = new THREE.Mesh(
  new THREE.CircleGeometry(10,45,0,6.4), //radius, segments, thetaStart, thetaLength
  new THREE.MeshBasicMaterial({color:0x00ff00, side: THREE.DoubleSide})
)
//scene.add(cerchio)

//CONE GEOMETRY
const cono = new THREE.Mesh(
  new THREE.ConeGeometry(1,2,56),
  new THREE.MeshBasicMaterial({color:0xff0000})
)
//scene.add(cono)

//CYLINDER GEOMETRY
const cilindro = new THREE.Mesh(
  new THREE.CylinderGeometry(5,5,20,32),
  new THREE.MeshBasicMaterial({color:0x00ff00})
)
cilindro.position.set(0,0,-10)
//scene.add(cilindro)

//RING GEOMETRY
const anello = new THREE.Mesh(
  new THREE.RingGeometry(),
  new THREE.MeshBasicMaterial({color:0x0000ff, side: THREE.DoubleSide})
)
//scene.add(anello)

//TORUS GEOMETRY
const torus = new THREE.Mesh(
  new THREE.TorusGeometry(),
  new THREE.MeshBasicMaterial({color:0xff0000})
)
//scene.add(torus)

//TORUSKNOT GEOMETRY
const torusKnot = new THREE.Mesh(
  new THREE.TorusKnotGeometry(),
  new THREE.MeshBasicMaterial({color:0x00ff00})
)
//scene.add(torusKnot)

//DODECAHEDRON GEOMETRY
const dodecahedron = new THREE.Mesh(
  new THREE.DodecahedronGeometry(),
  new THREE.MeshBasicMaterial({color:0x0000ff})
)
//scene.add(dodecahedron)

//OCTAHEDRON GEOMETRY
const octahedron = new THREE.Mesh(
  new THREE.OctahedronGeometry(),
  new THREE.MeshBasicMaterial({color:0x0000ff})
)
//scene.add(octahedron)

//TETRAHEDRON GEOMETRY
const tetrahedron = new THREE.Mesh(
  new THREE.TetrahedronGeometry(),
  new THREE.MeshBasicMaterial({color:0x0000ff})
)
//scene.add(tetrahedron)

//ICOSAHEDRON GEOMETRY
const icosahedron = new THREE.Mesh(
  new THREE.IcosahedronGeometry(),
  new THREE.MeshBasicMaterial({color:0x0000ff})
)
//scene.add(icosahedron)

//SPHERE GEOMETRY
const sphere = new THREE.Mesh(
  new THREE.SphereGeometry(),
  new THREE.MeshBasicMaterial({color:0x0000ff})
)
//scene.add(sphere)

//SHAPE GEOMETRY
const shape = new THREE.Mesh(
  new THREE.ShapeGeometry(),
  new THREE.MeshBasicMaterial({color:0x0000ff, side: THREE.DoubleSide})
)
//scene.add(shape)

// TUBE GEOMETRY
const tube = new THREE.Mesh(
  new THREE.TubeGeometry(),
  new THREE.MeshBasicMaterial({color:0x0000ff})
)
//scene.add(tube)

//EXTRUDE GEOMETRY
const extrude = new THREE.Mesh(
  new THREE.ExtrudeGeometry(),
  new THREE.MeshBasicMaterial({color:0x0000ff})
)
//scene.add(extrude)

//LATHE GEOMETRY usato per creare bicchieri , anfore e varie, crei un segmento e lo ripete in modo circolare
const lathe = new THREE.Mesh(
  new THREE.LatheGeometry(),
  new THREE.MeshBasicMaterial({color:0xff0000})
)
//scene.add(lathe)

//TEXT GEOMETRY


//combinando queste grometrie possiamo creare altri oggetti complesse (ad esempio case e altri oggetti)

// CAMERA
let camera = new THREE.PerspectiveCamera(
  75, 
  sizes.width / sizes.height, 
  0.1,
  100  
);
camera.position.set(0,0,3)
camera.lookAt(box.position)
scene.add(camera);

const clock = new THREE.Clock()

const animate = () =>{
  const elapsedTime = clock.getElapsedTime()

  controls.update()
  
  renderer.render(scene, camera)
}

onMounted(() => {
  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value
  });
 
  renderer.setSize(sizes.width, sizes.height)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio,2)) // in questo modo il max valore di pixel ratio utilizzaro è 2 (altrimenti devi renderizzare troppi pixel, dispiego enorme di potenza gpu)
  renderer.render(scene, camera)
  renderer.setAnimationLoop(animate)

  //CONTROLS
  controls = new OrbitControls(camera, canvasRef.value)
  controls.enableDamping = true; // permette uno effetto smooth una volta che rilasciamo l'elemento, rallenta fino a fermarsi
});

onUnmounted(() => {
  renderer.setAnimationLoop(null)
});

</script>

<style>
canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style>