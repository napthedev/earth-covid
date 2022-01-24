<script>
  import { onMount } from "svelte";
  import * as THREE from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  import { countries } from "../shared/constant";
  import {
    calcPositionFromCoordinates,
    calculateCreatedTime,
  } from "../shared/util";

  export let image;
  export let data;

  let container;

  let countryName = "";
  let total = 0;
  let death = 0;
  let isPopupOpened = false;
  let mouse = { x: undefined, y: undefined };

  onMount(() => {
    const camera = new THREE.PerspectiveCamera(
      75,
      innerWidth / innerHeight,
      0.1,
      1000
    );

    camera.position.z = 25;

    const scene = new THREE.Scene();
    const renderer = new THREE.WebGLRenderer();
    container.appendChild(renderer.domElement);

    renderer.setSize(innerWidth, innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);

    const sphereGeometry = new THREE.SphereGeometry(10, 100, 100);
    const sphereMaterial = new THREE.MeshPhongMaterial({
      map: new THREE.TextureLoader().load(image),
    });
    const sphereMesh = new THREE.Mesh(sphereGeometry, sphereMaterial);
    scene.add(sphereMesh);

    [...new Array(100)].forEach(() => {
      const starGeometry = new THREE.SphereGeometry(0.15);
      const starMaterial = new THREE.MeshPhongMaterial({
        color: 0xffffff,
      });
      const starMesh = new THREE.Mesh(starGeometry, starMaterial);

      const minDistance = 30;
      const maxDistance = 100;

      const [x, y, z] = [...new Array(3)].map(() =>
        Math.random() >= 0.5
          ? Math.floor(Math.random() * (maxDistance - minDistance)) +
            minDistance
          : -(
              Math.floor(Math.random() * (maxDistance - minDistance)) +
              minDistance
            )
      );
      starMesh.position.set(x, y, z);
      scene.add(starMesh);
    });

    const addCountryDot = (lat, lon, color, radius, opacity) => {
      const pos = calcPositionFromCoordinates(lat, lon, 10);

      const dotGeometry = new THREE.SphereGeometry(radius);
      const dotMaterial = new THREE.MeshPhongMaterial({
        color: new THREE.Color(color),
        opacity,
        transparent: true,
      });
      const dotMesh = new THREE.Mesh(dotGeometry, dotMaterial);
      dotMesh.position.x = pos.x;
      dotMesh.position.y = pos.y;
      dotMesh.position.z = pos.z;
      scene.add(dotMesh);

      return dotMesh;
    };

    const dots = [];

    data.Countries.forEach((country) => {
      const coordinates = countries[country.CountryCode];

      const configs =
        country.TotalConfirmed < data.Max / 200
          ? {
              color: "#00ff00",
              radius: 0.05,
              opacity: 0.5,
            }
          : country.TotalConfirmed < data.Max / 3
          ? {
              color: "#ffff00",
              radius: 0.1,
              opacity: 0.7,
            }
          : {
              color: "#ff0000",
              radius: 0.2,
              opacity: 0.9,
            };

      if (coordinates) {
        dots.push({
          ...configs,
          countryName: country.Country,
          total: country.TotalConfirmed,
          death: country.TotalDeaths,
          mesh: addCountryDot(
            coordinates.lat,
            coordinates.lon,
            configs.color,
            configs.radius,
            configs.opacity
          ),
        });
      }
    });

    const dotMesh = dots.map((dot) => dot.mesh);

    const ambientLight = new THREE.AmbientLight(0xffffff, 1);
    scene.add(ambientLight);

    const orbit = new OrbitControls(camera, renderer.domElement);
    orbit.maxDistance = 100;
    orbit.minDistance = 15;

    const raycaster = new THREE.Raycaster();
    const raycasterMouse = new THREE.Vector2(-999, -999);

    renderer.domElement.addEventListener("mousemove", (e) => {
      raycasterMouse.x = (e.clientX / window.innerWidth) * 2 - 1;
      raycasterMouse.y = -(e.clientY / window.innerHeight) * 2 + 1;

      mouse = { x: e.clientX, y: e.clientY };
    });

    const animate = () => {
      requestAnimationFrame(animate);

      raycaster.setFromCamera(raycasterMouse, camera);

      const intersects = raycaster.intersectObjects(dotMesh);

      if (intersects.length === 0) isPopupOpened = false;

      dots.forEach((dot) => {
        if (intersects[0] && dot.mesh === intersects[0].object) {
          dot.mesh.material.opacity = 1;
          total = dot.total;
          death = dot.death;
          countryName = dot.countryName;
          isPopupOpened = true;
        } else {
          dot.mesh.material.opacity = dot.opacity;
        }
      });

      renderer.render(scene, camera);
    };

    animate();

    window.addEventListener("resize", () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();

      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  });
</script>

<main bind:this={container}>
  <div
    class="pop-up"
    style="opacity: {isPopupOpened ? '1' : '0'}; visibility: {isPopupOpened
      ? 'visible'
      : 'invisible'}; top: {mouse.y}px; left: {mouse.x}px;"
  >
    <p>{countryName}</p>
    <p style="font-size: 20px;">Total: {total.toLocaleString()}</p>
    <p style="font-size: 14px;">Death: {death.toLocaleString()}</p>
  </div>

  <p style="position: absolute; top: 10px; left: 10px;">Earth's Covid Status</p>
  <p style="position: absolute; top: 10px; right: 10px">
    Updated: {calculateCreatedTime(new Date(data.Date).getTime())}
  </p>
  <p style="position: absolute; bottom: 10px; left: 10px">
    Total: {data.Global.TotalConfirmed.toLocaleString()}
  </p>
  <p style="position: absolute; bottom: 10px; right: 10px">
    Deaths: {data.Global.TotalDeaths.toLocaleString()}
  </p>
</main>

<style>
  .pop-up {
    position: absolute;
    background: #222222c7;
    padding: 10px;
    border-radius: 5px;
    transition: opacity 0.15s;
    pointer-events: none;
    user-select: none;
  }
</style>
