<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Agenda 2030: Eco-Esquive 3D</title>
    <style>
        * { box-sizing: border-box; user-select: none; margin: 0; padding: 0; }
        body {
            background: #05070a;
            color: white;
            font-family: 'Segoe UI', system-ui, sans-serif;
            overflow: hidden;
            height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        #canvas-container {
            position: relative;
            width: 100vw;
            height: 100vh;
        }
        canvas { display: block; width: 100%; height: 100%; }

        .overlay {
            position: absolute;
            inset: 0;
            background: rgba(10, 14, 24, 0.85);
            backdrop-filter: blur(6px);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 10;
            padding: 20px;
            text-align: center;
        }
        .oculto { display: none !important; }

        h1 {
            font-size: 2.7rem;
            text-transform: uppercase;
            background: linear-gradient(90deg, #1dd1a1, #48dbfb, #feca57);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            margin-bottom: 8px;
            letter-spacing: 2px;
        }
        p.sub { color: #c8d6e5; max-width: 680px; margin-bottom: 18px; font-size: 0.95rem; }

        .btn {
            background: linear-gradient(135deg, #1dd1a1, #10ac84);
            color: #0b0f19;
            border: 2px solid #fff;
            padding: 14px 30px;
            font-size: 1.1rem;
            font-weight: 900;
            border-radius: 12px;
            cursor: pointer;
            margin: 8px;
            text-transform: uppercase;
            box-shadow: 0 5px 15px rgba(29, 209, 161, 0.4);
            transition: transform 0.15s;
        }
        .btn:hover { transform: scale(1.06); }
        .btn-azul {
            background: linear-gradient(135deg, #48dbfb, #0abde3);
            box-shadow: 0 5px 15px rgba(72, 219, 251, 0.4);
        }

        .grid-ods {
            display: grid;
            grid-template-columns: repeat(6, 1fr);
            gap: 8px;
            max-width: 860px;
            width: 100%;
            margin-bottom: 18px;
        }
        .card-ods {
            padding: 9px 5px;
            border-radius: 8px;
            cursor: pointer;
            border: 2px solid rgba(255,255,255,0.25);
            transition: 0.15s;
            font-weight: bold;
            text-shadow: 1px 1px 2px #000;
        }
        .card-ods:hover { transform: scale(1.05); border-color: #fff; }
        .card-ods.activa { border-color: #feca57; box-shadow: 0 0 15px #feca57; transform: scale(1.06); }
        .card-ods .num { font-size: 1.05rem; }
        .card-ods .txt { font-size: 0.68rem; margin-top: 2px; }

        .grid-naves {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 15px;
            max-width: 700px;
            width: 100%;
            margin-bottom: 20px;
        }
        .card-nave {
            background: rgba(255,255,255,0.08);
            border: 2px solid rgba(255,255,255,0.2);
            border-radius: 12px;
            padding: 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            text-align: left;
        }

        #hud-3d {
            position: absolute;
            top: 20px;
            left: 30px;
            right: 30px;
            display: flex;
            justify-content: space-between;
            font-size: 1.15rem;
            font-weight: 900;
            z-index: 5;
            pointer-events: none;
            text-shadow: 2px 2px 4px #000;
        }
        .hud-caja {
            background: rgba(0, 0, 0, 0.6);
            padding: 10px 20px;
            border-radius: 10px;
            border: 1px solid rgba(255, 255, 255, 0.25);
        }
        .ayuda-controles {
            position: absolute;
            bottom: 18px;
            width: 100%;
            text-align: center;
            color: #fff;
            font-size: 0.95rem;
            font-weight: bold;
            pointer-events: none;
            z-index: 5;
            text-shadow: 1px 1px 4px #000;
        }
    </style>
</head>
<body>

<div id="canvas-container">
    <div id="hud-3d" class="oculto">
        <div class="hud-caja">🌍 <span id="hud-ods">ODS 1</span> | ❤️ Escudo: <span id="hud-vida" style="color:#1dd1a1;">100%</span></div>
        <div class="hud-caja">⚡ Puntos: <span id="hud-puntos" style="color:#48dbfb;">0</span></div>
        <div class="hud-caja">🪙 Eco-Monedas: <span id="hud-monedas" style="color:#feca57;">0</span></div>
    </div>

    <div id="ayuda-controles" class="ayuda-controles oculto">
        ⬅️ ➡️ Cambiar Carril | ⬆️ o ESPACIO Saltar | SHIFT Turbo
    </div>

    <div id="menu-principal" class="overlay">
        <h1>🚀 AGENDA 2030: ECO-ESQUIVE 3D</h1>
        <p class="sub">Elige un ODS, esquiva los obstáculos y recoge orbes de energía limpia.</p>
        
        <div class="grid-ods" id="contenedor-ods"></div>

        <div>
            <button class="btn" id="btn-despegar">▶ INICIAR MISIÓN (<span id="txt-ods-sel">ODS 1</span>)</button>
            <button class="btn btn-azul" id="btn-hangar">🛸 HANGAR DE NAVES (🪙 <span id="monedas-menu">0</span>)</button>
        </div>
    </div>

    <div id="pantalla-hangar" class="overlay oculto">
        <h1>🛸 HANGAR DE NAVES 3D</h1>
        <p class="sub">Tus Eco-Monedas: <strong style="color:#feca57;">🪙 <span id="monedas-hangar">0</span></strong></p>
        <div class="grid-naves" id="contenedor-naves"></div>
        <button class="btn" id="btn-cerrar-hangar">VOLVER AL MENÚ</button>
    </div>

    <div id="pantalla-gameover" class="overlay oculto">
        <h1 style="background: linear-gradient(90deg, #ff4757, #ff9f43); -webkit-background-clip: text;">¡IMPACTO RECIBIDO!</h1>
        <p class="sub" id="resumen-final" style="font-size: 1.2rem; color: #fff;"></p>
        <div>
            <button class="btn" id="btn-reintentar">🔄 VOLVER A INTENTAR</button>
            <button class="btn btn-azul" id="btn-menu-fin">🏠 MENÚ PRINCIPAL</button>
        </div>
    </div>
</div>

<script src="https://unpkg.com/three@0.128.0/build/three.min.js"></script>
<script>
window.addEventListener("DOMContentLoaded", () => {

    const ODS_3D = [
        { id: 1, nombre: "Fin Pobreza", color: "#e5243b", hex: 0xe5243b },
        { id: 2, nombre: "Hambre Cero", color: "#dda63a", hex: 0xdda63a },
        { id: 3, nombre: "Salud", color: "#4c9f38", hex: 0x4c9f38 },
        { id: 4, nombre: "Educación", color: "#c5192d", hex: 0xc5192d },
        { id: 5, nombre: "Igualdad", color: "#ff3a21", hex: 0xff3a21 },
        { id: 6, nombre: "Agua Limpia", color: "#26bde2", hex: 0x26bde2 },
        { id: 7, nombre: "Energía Limpia", color: "#fcc30b", hex: 0xfcc30b },
        { id: 8, nombre: "Trabajo Decente", color: "#a21942", hex: 0xa21942 },
        { id: 9, nombre: "Innovación", color: "#fd6925", hex: 0xfd6925 },
        { id: 10, nombre: "Desigualdades", color: "#dd1367", hex: 0xdd1367 },
        { id: 11, nombre: "Ciudades", color: "#fd9d24", hex: 0xfd9d24 },
        { id: 12, nombre: "Producción", color: "#bf8b2e", hex: 0xbf8b2e },
        { id: 13, nombre: "Clima", color: "#3f7e44", hex: 0x3f7e44 },
        { id: 14, nombre: "Vida Submarina", color: "#0a97d9", hex: 0x0a97d9 },
        { id: 15, nombre: "Vida Terrestre", color: "#56c02b", hex: 0x56c02b },
        { id: 16, nombre: "Paz y Justicia", color: "#00689d", hex: 0x00689d },
        { id: 17, nombre: "Alianzas", color: "#19486a", hex: 0x19486a }
    ];

    const NAVES_3D = [
        { id: 'caza_eco', nombre: 'Eco-Nave Ágil', desc: 'Rápida y equilibrada.', precio: 0, color: 0x1dd1a1, tipo: 'cono' },
        { id: 'interceptor_solar', nombre: 'Interceptor Solar', desc: 'Alas delta doradas.', precio: 150, color: 0xfeca57, tipo: 'piramide' },
        { id: 'esfera_gaia', nombre: 'Orbe Cuántico', desc: 'Tecnología avanzada.', precio: 300, color: 0x48dbfb, tipo: 'octaedro' },
        { id: 'titan_verde', nombre: 'Crucero Pesado', desc: 'Blindaje extra.', precio: 500, color: 0xa29bfe, tipo: 'cubo' }
    ];

    let datosJugador = JSON.parse(localStorage.getItem("ecoEsquive3D")) || {
        monedas: 200,
        naveEquipada: 'caza_eco',
        navesCompradas: ['caza_eco']
    };

    function guardarDatos() {
        localStorage.setItem("ecoEsquive3D", JSON.stringify(datosJugador));
        document.getElementById("monedas-menu").innerText = datosJugador.monedas;
        document.getElementById("monedas-hangar").innerText = datosJugador.monedas;
        document.getElementById("hud-monedas").innerText = datosJugador.monedas;
    }

    if (typeof THREE === "undefined") {
        alert("Error cargando Three.js. Comprueba tu conexión.");
        return;
    }

    const contenedor = document.getElementById("canvas-container");
    const escena = new THREE.Scene();
    escena.fog = new THREE.FogExp2(0x05070a, 0.014);

    const camara = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 200);
    camara.position.set(0, 4.5, 9);
    camara.lookAt(0, 1, -10);

    const renderizador = new THREE.WebGLRenderer({ antialias: true });
    renderizador.setSize(window.innerWidth, window.innerHeight);
    contenedor.appendChild(renderizador.domElement);

    const luzAmbiental = new THREE.AmbientLight(0xffffff, 0.75);
    escena.add(luzAmbiental);

    const luzDireccional = new THREE.DirectionalLight(0xffffff, 0.9);
    luzDireccional.position.set(10, 20, 10);
    escena.add(luzDireccional);

    const luzNave = new THREE.PointLight(0x1dd1a1, 2.5, 18);
    escena.add(luzNave);

    const suelo = new THREE.Mesh(
        new THREE.PlaneGeometry(14, 250),
        new THREE.MeshStandardMaterial({ color: 0x0f141f, roughness: 0.4 })
    );
    suelo.rotation.x = -Math.PI / 2;
    suelo.position.z = -100;
    escena.add(suelo);

    const gridHelper = new THREE.GridHelper(14, 28, 0x1dd1a1, 0x1e272e);
    gridHelper.position.set(0, 0.02, -100);
    gridHelper.scale.z = 18;
    escena.add(gridHelper);

    let decoraciones = [];
    function crearDecoraciones3D(colorHex) {
        decoraciones.forEach(d => escena.remove(d));
        decoraciones = [];
        const geo = new THREE.ConeGeometry(3, 10, 4);
        const mat = new THREE.MeshStandardMaterial({ color: colorHex, roughness: 0.4, flatShading: true });

        for (let i = 0; i < 30; i++) {
            const mIzq = new THREE.Mesh(geo, mat);
            mIzq.position.set(-15, 4, -i * 8);
            escena.add(mIzq);
            decoraciones.push(mIzq);

            const mDer = new THREE.Mesh(geo, mat);
            mDer.position.set(15, 4, -i * 8);
            escena.add(mDer);
            decoraciones.push(mDer);
        }
    }

    let naveMesh;
    function construirNave3D() {
        if (naveMesh) escena.remove(naveMesh);
        const info = NAVES_3D.find(n => n.id === datosJugador.naveEquipada);

        let geo;
        if (info.tipo === 'cono') geo = new THREE.ConeGeometry(0.85, 2.5, 6);
        else if (info.tipo === 'piramide') geo = new THREE.ConeGeometry(1.15, 2.3, 4);
        else if (info.tipo === 'octaedro') geo = new THREE.OctahedronGeometry(0.95);
        else geo = new THREE.BoxGeometry(1.4, 0.85, 2.1);

        const mat = new THREE.MeshStandardMaterial({
            color: info.color,
            emissive: info.color,
            emissiveIntensity: 0.4,
            roughness: 0.2,
            metalness: 0.7
        });

        naveMesh = new THREE.Mesh(geo, mat);
        if (info.tipo === 'cono' || info.tipo === 'piramide') {
            naveMesh.rotation.x = -Math.PI / 2;
        }
        naveMesh.position.set(0, 1.2, 2);
        escena.add(naveMesh);
        luzNave.color.setHex(info.color);
    }

    let odsSeleccionado = 1;
    let jugando = false;
    let carrilActual = 0;
    const CARRILES = [-3.5, 0, 3.5];

    let velocidadY = 0;
    let enSuelo = true;
    let escudo = 100;
    let puntos = 0;
    let velocidadMundo = 0.9;
    let turboActivo = false;

    let obstaculos = [];
    let orbes = [];

    function actualizarEntornoODS() {
        const ods = ODS_3D.find(o => o.id === odsSeleccionado);
        gridHelper.material.color.setHex(ods.hex);
        escena.fog.color.setHex(ods.hex);
        crearDecoraciones3D(ods.hex);
    }

    function renderizarMenuODS() {
        const cont = document.getElementById("contenedor-ods");
        cont.innerHTML = "";
        ODS_3D.forEach(ods => {
            const div = document.createElement("div");
            div.className = "card-ods" + (ods.id === odsSeleccionado ? " activa" : "");
            div.style.backgroundColor = ods.color;
            div.innerHTML = `<div class="num">ODS ${ods.id}</div><div class="txt">${ods.nombre}</div>`;
            div.addEventListener("click", () => {
                odsSeleccionado = ods.id;
                document.querySelectorAll(".card-ods").forEach(c => c.classList.remove("activa"));
                div.classList.add("activa");
                document.getElementById("txt-ods-sel").innerText = `ODS ${ods.id}`;
                actualizarEntornoODS();
            });
            cont.appendChild(div);
        });
    }

    function abrirHangar() {
        document.getElementById("menu-principal").classList.add("oculto");
        document.getElementById("pantalla-hangar").classList.remove("oculto");
        const cont = document.getElementById("contenedor-naves");
        cont.innerHTML = "";

        NAVES_3D.forEach(nave => {
            const comprada = datosJugador.navesCompradas.includes(nave.id);
            const equipada = datosJugador.naveEquipada === nave.id;
            const div = document.createElement("div");
            div.className = "card-nave";

            const btn = document.createElement("button");
            btn.className = "btn" + (equipada ? " btn-azul" : "");
            btn.style.padding = "8px 14px";
            btn.style.fontSize = "0.85rem";

            if (!comprada) {
                btn.innerText = `🪙 ${nave.precio}`;
                btn.onclick = () => {
                    if (datosJugador.monedas >= nave.precio) {
                        datosJugador.monedas -= nave.precio;
                        datosJugador.navesCompradas.push(nave.id);
                        datosJugador.naveEquipada = nave.id;
                        guardarDatos();
                        construirNave3D();
                        abrirHangar();
                    } else {
                        alert("¡Necesitas más Eco-Monedas!");
                    }
                };
            } else if (equipada) {
                btn.innerText = "EQUIPADA";
                btn.style.cursor = "default";
            } else {
                btn.innerText = "USAR";
                btn.onclick = () => {
                    datosJugador.naveEquipada = nave.id;
                    guardarDatos();
                    construirNave3D();
                    abrirHangar();
                };
            }

            div.innerHTML = `<div><strong style="color:#1dd1a1;">${nave.nombre}</strong><div style="font-size:0.8rem; color:#ccc;">${nave.desc}</div></div>`;
            div.appendChild(btn);
            cont.appendChild(div);
        });
    }

    function iniciarJuego3D() {
        document.getElementById("menu-principal").classList.add("oculto");
        document.getElementById("pantalla-gameover").classList.add("oculto");
        document.getElementById("hud-3d").classList.remove("oculto");
        document.getElementById("ayuda-controles").classList.remove("oculto");

        obstaculos.forEach(o => escena.remove(o));
        orbes.forEach(e => escena.remove(e));
        obstaculos = []; orbes = [];

        carrilActual = 0;
        naveMesh.position.set(0, 1.2, 2);
        naveMesh.rotation.set(-Math.PI / 2, 0, 0);
        velocidadY = 0;
        enSuelo = true;
        escudo = 100;
        puntos = 0;
        velocidadMundo = 0.9;
        jugando = true;

        const ods = ODS_3D.find(o => o.id === odsSeleccionado);
        document.getElementById("hud-ods").innerText = `ODS ${ods.id}: ${ods.nombre}`;
        actualizarHUD();
    }

    function actualizarHUD() {
        document.getElementById("hud-vida").innerText = Math.max(0, escudo) + "%";
        document.getElementById("hud-puntos").innerText = Math.floor(puntos);
        document.getElementById("hud-monedas").innerText = datosJugador.monedas;
    }

    document.getElementById("btn-despegar").addEventListener("click", iniciarJuego3D);
    document.getElementById("btn-reintentar").addEventListener("click", iniciarJuego3D);
    document.getElementById("btn-hangar").addEventListener("click", abrirHangar);
    document.getElementById("btn-cerrar-hangar").addEventListener("click", () => {
        document.getElementById("pantalla-hangar").classList.add("oculto");
        document.getElementById("menu-principal").classList.remove("oculto");
    });
    document.getElementById("btn-menu-fin").addEventListener("click", () => {
        document.getElementById("pantalla-gameover").classList.add("oculto");
        document.getElementById("hud-3d").classList.add("oculto");
        document.getElementById("ayuda-controles").classList.add("oculto");
        document.getElementById("menu-principal").classList.remove("oculto");
    });

    window.addEventListener("keydown", e => {
        if (!jugando) return;
        if ((e.code === "ArrowLeft" || e.code === "KeyA") && carrilActual > -1) carrilActual--;
        if ((e.code === "ArrowRight" || e.code === "KeyD") && carrilActual < 1) carrilActual++;
        if ((e.code === "ArrowUp" || e.code === "Space" || e.code === "KeyW") && enSuelo) {
            velocidadY = 0.32;
            enSuelo = false;
        }
        if (e.code === "ShiftLeft" || e.code === "ShiftRight") turboActivo = true;
    });

    window.addEventListener("keyup", e => {
        if (e.code === "ShiftLeft" || e.code === "ShiftRight") turboActivo = false;
    });

    setInterval(() => {
        if (!jugando) return;
        const posX = CARRILES[Math.floor(Math.random() * 3)];

        if (Math.random() < 0.55) {
            const obs = new THREE.Mesh(
                new THREE.BoxGeometry(2.2, 1.8, 1.8),
                new THREE.MeshStandardMaterial({ color: 0xff4757, roughness: 0.3 })
            );
            obs.position.set(posX, 0.9, -110);
            escena.add(obs);
            obstaculos.push(obs);
        } else {
            const orbe = new THREE.Mesh(
                new THREE.OctahedronGeometry(0.85),
                new THREE.MeshStandardMaterial({ color: 0x1dd1a1, emissive: 0x1dd1a1, emissiveIntensity: 0.6 })
            );
            orbe.position.set(posX, 1.3, -110);
            escena.add(orbe);
            orbes.push(orbe);
        }
    }, 580);

    function animar() {
        requestAnimationFrame(animar);

        const velActual = jugando ? (turboActivo ? velocidadMundo * 1.7 : velocidadMundo) : 0.25;
        gridHelper.position.z += velActual;
        if (gridHelper.position.z > -50) gridHelper.position.z = -100;

        decoraciones.forEach(d => {
            d.position.z += velActual;
            if (d.position.z > 15) d.position.z = -220;
        });

        if (!jugando) {
            if (naveMesh) {
                naveMesh.position.y = 1.6 + Math.sin(Date.now() * 0.003) * 0.3;
                naveMesh.rotation.y += 0.02;
            }
            renderizador.render(escena, camara);
            return;
        }

        const destinoX = CARRILES[carrilActual + 1];
        naveMesh.position.x += (destinoX - naveMesh.position.x) * 0.18;
        naveMesh.rotation.z = (naveMesh.position.x - destinoX) * 0.25;

        naveMesh.position.y += velocidadY;
        if (naveMesh.position.y > 1.2) {
            velocidadY -= 0.015;
        } else {
            naveMesh.position.y = 1.2;
            velocidadY = 0;
            enSuelo = true;
        }

        luzNave.position.copy(naveMesh.position);
        luzNave.position.y += 1.5;

        puntos += turboActivo ? 0.6 : 0.3;
        velocidadMundo = Math.min(1.7, 0.9 + (puntos / 2500));

        for (let i = obstaculos.length - 1; i >= 0; i--) {
            const obs = obstaculos[i];
            obs.position.z += velActual;

            if (obs.position.distanceTo(naveMesh.position) < 1.5) {
                escudo -= 35;
                escena.remove(obs);
                obstaculos.splice(i, 1);
                if (escudo <= 0) {
                    jugando = false;
                    document.getElementById("hud-3d").classList.add("oculto");
                    document.getElementById("ayuda-controles").classList.add("oculto");
                    document.getElementById("pantalla-gameover").classList.remove("oculto");
                    document.getElementById("resumen-final").innerText = `Puntuación: ${Math.floor(puntos)} | Eco-Monedas: 🪙 ${datosJugador.monedas}`;
                    return;
                }
                continue;
            }

            if (obs.position.z > 15) {
                escena.remove(obs);
                obstaculos.splice(i, 1);
            }
        }

        for (let i = orbes.length - 1; i >= 0; i--) {
            const orbe = orbes[i];
            orbe.position.z += velActual;
            orbe.rotation.y += 0.06;

            if (orbe.position.distanceTo(naveMesh.position) < 1.8) {
                puntos += 60;
                datosJugador.monedas += 15;
                escudo = Math.min(100, escudo + 15);
                guardarDatos();
                escena.remove(orbe);
                orbes.splice(i, 1);
                continue;
            }

            if (orbe.position.z > 15) {
                escena.remove(orbe);
                orbes.splice(i, 1);
            }
        }

        actualizarHUD();
        renderizador.render(escena, camara);
    }

    window.addEventListener("resize", () => {
        camara.aspect = window.innerWidth / window.innerHeight;
        camara.updateProjectionMatrix();
        renderizador.setSize(window.innerWidth, window.innerHeight);
    });

    guardarDatos();
    renderizarMenuODS();
    actualizarEntornoODS();
    construirNave3D();
    animar();
});
</script>
</body>
</html>
