<script lang="ts">
  // Usamos tipado flexible para evitar que TypeScript bloquee el renderizado en el playground
  let seccion: any = $state('inicio');
  let sesion = $state(false);
  let correo = $state('');
  let contrasena = $state('');
  let error = $state('');
  let busqueda = $state('');
  let modalEquipo = $state(false);
  let modalSolicitud = $state(false);
  let modalPerdido = $state(false);
  let mensaje = $state('');

  let equipos = $state([
    { id: 1, codigo: 'AP-001', nombre: 'Laptop Dell Latitude', usuario: 'María Santos', sucursal: 'Naco', condicion: 'Bueno', estado: 'Asignado' },
    { id: 2, codigo: 'AP-002', nombre: 'Laptop HP ProBook', usuario: 'Carlos Pérez', sucursal: 'Santiago', condicion: 'Dañado', estado: 'En reparación' },
    { id: 3, codigo: 'AP-003', nombre: 'Monitor Dell', usuario: 'Sin asignar', sucursal: 'Principal', condicion: 'Bueno', estado: 'Disponible' },
    { id: 4, codigo: 'AP-004', nombre: 'Laptop Lenovo', usuario: 'Ana Gómez', sucursal: 'Piantini', condicion: 'Obsoleto', estado: 'Evaluar sustitución' }
  ]);

  let solicitudes = $state([
    { id: 1001, asunto: 'Laptop no enciende', usuario: 'Carlos Pérez', tipo: 'Reparación', prioridad: 'Alta', estado: 'En proceso' },
    { id: 1002, asunto: 'Solicitar equipo sustituto', usuario: 'Ana Gómez', tipo: 'Sustitución', prioridad: 'Media', estado: 'Pendiente' }
  ]);

  let perdidos = $state([
    { nombre: 'Adaptador USB-C', lugar: 'Sala de reuniones, piso 2', fecha: '2026-10-14', estado: 'En custodia' },
    { nombre: 'Cargador de laptop', lugar: 'Sucursal Naco', fecha: '2026-10-13', estado: 'Pendiente de identificar' }
  ]);

  let equipoNuevo = $state({
    codigo: '', nombre: '', usuario: '', sucursal: '',
    condicion: 'Bueno', estado: 'Disponible'
  });

  let solicitudNueva = $state({
    asunto: '', usuario: '', tipo: 'Reparación', prioridad: 'Media'
  });

  let perdidoNuevo = $state({
    nombre: '', lugar: '', fecha: '', estado: 'Pendiente de identificar'
  });

  const menu = [
    { id: 'inicio', nombre: 'Resumen', icono: '⌂' },
    { id: 'equipos', nombre: 'Inventario de equipos', icono: '▣' },
    { id: 'solicitudes', nombre: 'Solicitudes técnicas', icono: '▤' },
    { id: 'perdidos', nombre: 'Objetos perdidos', icono: '⌕' }
  ];

  let equiposFiltrados = $derived(
    equipos.filter((e: any) =>
      Object.values(e).join(' ').toLowerCase()
        .includes(busqueda.toLowerCase())
    )
  );

  let solicitudesFiltradas = $derived(
    solicitudes.filter((s: any) =>
      Object.values(s).join(' ').toLowerCase()
        .includes(busqueda.toLowerCase())
    )
  );

  let totalBuenos = $derived(
    equipos.filter((e: any) => e.condicion === 'Bueno').length
  );

  let totalReparacion = $derived(
    equipos.filter((e: any) => e.estado === 'En reparación').length
  );

  let totalSustitucion = $derived(
    equipos.filter((e: any) => e.estado === 'Evaluar sustitución').length
  );

  function notificar(texto: string) {
    mensaje = texto;
    setTimeout(() => mensaje = '', 3000);
  }

  function iniciarSesion(event: SubmitEvent) {
    event.preventDefault();
    error = '';

    if (!correo.includes('@') || contrasena.length < 4) {
      error = 'Introduce un correo válido y una contraseña de al menos 4 caracteres.';
      return;
    }

    sesion = true;
    notificar('Entraste en la demostración');
  }

  function guardarEquipo(event: SubmitEvent) {
    event.preventDefault();

    if (equipos.some((e: any) => e.codigo === equipoNuevo.codigo)) {
      notificar('Ese código de equipo ya existe.');
      return;
    }

    equipos.unshift({
      ...equipoNuevo,
      id: Date.now()
    });

    equipoNuevo = {
      codigo: '', nombre: '', usuario: '', sucursal: '',
      condicion: 'Bueno', estado: 'Disponible'
    };

    modalEquipo = false;
    notificar('Equipo registrado en la demostración.');
  }

  function guardarSolicitud(event: SubmitEvent) {
    event.preventDefault();

    solicitudes.unshift({
      ...solicitudNueva,
      id: Math.max(1000, ...solicitudes.map((s: any) => s.id)) + 1,
      estado: 'Pendiente'
    });

    solicitudNueva = {
      asunto: '', usuario: '', tipo: 'Reparación', prioridad: 'Media'
    };

    modalSolicitud = false;
    notificar('Solicitud registrada en la demostración.');
  }

  function guardarPerdido(event: SubmitEvent) {
    event.preventDefault();

    perdidos.unshift({
      ...perdidoNuevo
    });

    perdidoNuevo = {
      nombre: '', lugar: '', fecha: '',
      estado: 'Pendiente de identificar'
    };

    modalPerdido = false;
    notificar('Objeto registrado en la demostración.');
  }

  function abrir(seccionNueva: any) {
    seccion = seccionNueva;
    busqueda = '';
  }
</script>

<svelte:head>

  <title>Centro de Soporte | Portal interno</title>
  <meta name="description" content="Gestión interna de equipos y soporte técnico." />
</svelte:head>

{#if !sesion}

  <main class="login">
    <section class="login-brand">
      <div class="logo">
        <span class="logo-mark">a</span>
        <div>APAP<small>Soporte tecnológico</small></div>
      </div>

      <div class="brand-copy">
        <span class="eyebrow">PORTAL INTERNO</span>
        <h1>La tecnología que mantiene todo en movimiento.</h1>
        <p>Gestiona equipos, solicitudes de reparación, sustituciones y objetos encontrados desde un solo lugar.</p>
      </div>

      <small class="legal">Prototipo conceptual · No es un sistema oficial de APAP</small>
    </section>

    <section class="login-content">
      <form class="login-form" onsubmit={iniciarSesion}>
        <span class="eyebrow">BIENVENIDO/A</span>
        <h2>Iniciar sesión</h2>
        <p class="muted">Accede al centro de soporte tecnológico.</p>

        <label for="correo">Correo institucional</label>
        <input id="correo" type="email" bind:value={correo}
          placeholder="nombre@empresa.com" required />

        <label for="contrasena">Contraseña</label>
        <input id="contrasena" type="password" bind:value={contrasena}
          placeholder="Introduce tu contraseña" required />

        {#if error}
          <p class="error">{error}</p>
        {/if}

        <button class="primary full" type="submit">Entrar al portal →</button>

        <div class="security">
          <strong>Acceso restringido</strong>
          <p>Esta pantalla es una demostración. No introduzcas credenciales reales.</p>
        </div>
      </form>
    </section>

  </main>
{:else}
  <div class="app">
    <aside class="sidebar">
      <div class="logo sidebar-logo">
        <span class="logo-mark">a</span>
        <div>APAP<small>Soporte tecnológico</small></div>
      </div>

      <p class="nav-label">ESPACIO DE TRABAJO</p>

      <nav>
        {#each menu as item}
          <button
            class:active={seccion === item.id}
            class="nav-item"
            onclick={() => abrir(item.id)}>
            <span>{item.icono}</span>
            {item.nombre}
          </button>
        {/each}
      </nav>

      <div class="sidebar-bottom">
        <div class="help-card">
          <strong>Centro de ayuda</strong>
          <p>Gestiona las incidencias tecnológicas de tu organización.</p>
        </div>
        <button class="logout" onclick={() => sesion = false}>
          ↪ Cerrar sesión
        </button>
      </div>
    </aside>

    <main class="main">
      <header class="topbar">
        <span>Portal interno / <strong>{menu.find(m => m.id === seccion)?.nombre}</strong></span>
        <span class="online"><i></i> Vista de demostración</span>
      </header>

      <div class="content">
        {#if seccion === 'inicio'}
          <div class="heading">
            <div>
              <span class="eyebrow">CENTRO DE CONTROL</span>
              <h1>Resumen general</h1>
              <p class="muted">Estado del inventario y las solicitudes técnicas.</p>
            </div>
            <button class="primary" onclick={() => modalSolicitud = true}>
              + Nueva solicitud
            </button>
          </div>

          <section class="banner">
            <span class="eyebrow">PANEL DE SOPORTE</span>
            <h2>Todo bajo control.</h2>
            <p>Administra el ciclo de vida de los equipos y da seguimiento a cada solicitud.</p>
          </section>

          <section class="stats">
            <article class="stat">
              <span class="stat-icon blue">▣</span>
              <p>Equipos registrados</p>
              <strong>{equipos.length}</strong>
              <button onclick={() => abrir('equipos')}>Ver inventario →</button>
            </article>
            <article class="stat">
              <span class="stat-icon green">✓</span>
              <p>En buen estado</p>
              <strong>{totalBuenos}</strong>
              <small>Equipos operativos</small>
            </article>
            <article class="stat">
              <span class="stat-icon orange">↻</span>
              <p>En reparación</p>
              <strong>{totalReparacion}</strong>
              <button onclick={() => abrir('equipos')}>Revisar →</button>
            </article>
            <article class="stat">
              <span class="stat-icon purple">⇄</span>
              <p>Por sustituir</p>
              <strong>{totalSustitucion}</strong>
              <small>Requieren evaluación</small>
            </article>
          </section>

          <section class="panel">
            <div class="panel-heading">
              <div><h3>Solicitudes recientes</h3><p>Seguimiento de incidencias</p></div>
              <button class="text-button" onclick={() => abrir('solicitudes')}>Ver todas →</button>
            </div>

            {#each solicitudes.slice(0, 4) as s}
              <div class="list-row">
                <div class="row-icon">▤</div>
                <div class="row-main">
                  <strong>{s.asunto}</strong>
                  <small>#{s.id} · {s.usuario} · {s.tipo}</small>
                </div>
                <span class="pill" class:pending={s.estado === 'Pendiente'}
                  class:progress={s.estado === 'En proceso'}
                  class:resolved={s.estado === 'Resuelto'}>{s.estado}</span>
              </div>
            {/each}
          </section>

          <section class="panel">
            <div class="panel-heading"><div><h3>Accesos rápidos</h3><p>Acciones frecuentes</p></div></div>
            <div class="quick-links">
              <button onclick={() => { abrir('equipos'); modalEquipo = true; }}>＋ Registrar equipo</button>
              <button onclick={() => { abrir('solicitudes'); modalSolicitud = true; }}>＋ Crear solicitud</button>
              <button onclick={() => { abrir('perdidos'); modalPerdido = true; }}>＋ Registrar objeto perdido</button>
            </div>
          </section>

        {:else if seccion === 'equipos'}
          <div class="heading">
            <div>
              <span class="eyebrow">ACTIVOS TECNOLÓGICOS</span>
              <h1>Inventario de equipos</h1>
              <p class="muted">Equipos asignados, disponibles, en reparación o por sustituir.</p>
            </div>
            <button class="primary" onclick={() => modalEquipo = true}>+ Registrar equipo</button>
          </div>

          <div class="search">
            <span>⌕</span>
            <input bind:value={busqueda} placeholder="Buscar equipo, código, usuario o sucursal..." />
          </div>

          <section class="panel table-panel">
            <div class="table-wrap">
              <table>
                <thead><tr><th>CÓDIGO / EQUIPO</th><th>USUARIO</th><th>SUCURSAL</th><th>CONDICIÓN</th><th>ESTADO</th></tr></thead>
                <tbody>
                  {#each equiposFiltrados as e}
                    <tr>
                      <td><strong>{e.nombre}</strong><small>{e.codigo}</small></td>
                      <td>{e.usuario || 'Sin asignar'}</td>
                      <td>{e.sucursal || '—'}</td>
                      <td><span class="condition">{e.condicion}</span></td>
                      <td><span class="pill" class:pending={e.estado === 'Disponible'}
                        class:progress={e.estado === 'En reparación'}
                        class:replace={e.estado === 'Evaluar sustitución'}
                        class:resolved={e.estado === 'Asignado'}>{e.estado}</span></td>
                    </tr>
                  {/each}
                </tbody>
              </table>
            </div>
            {#if equiposFiltrados.length === 0}<p class="empty">No se encontraron equipos.</p>{/if}
            <p class="table-foot">{equiposFiltrados.length} equipos · Datos de demostración</p>
          </section>

          <div class="notice"><strong>Política de sustitución:</strong> los equipos obsoletos o con averías recurrentes deben pasar por evaluación técnica antes de aprobar su sustitución.</div>

        {:else if seccion === 'solicitudes'}
          <div class="heading">
            <div>
              <span class="eyebrow">MESA DE AYUDA</span>
              <h1>Solicitudes técnicas</h1>
              <p class="muted">Incidencias, reparaciones y solicitudes de sustitución.</p>
            </div>
            <button class="primary" onclick={() => modalSolicitud = true}>+ Nueva solicitud</button>
          </div>

          <div class="search">
            <span>⌕</span>
            <input bind:value={busqueda} placeholder="Buscar solicitud o usuario..." />
          </div>

          <section class="panel table-panel">
            <div class="table-wrap">
              <table>
                <thead><tr><th>N.º / ASUNTO</th><th>USUARIO</th><th>TIPO</th><th>PRIORIDAD</th><th>ESTADO</th></tr></thead>
                <tbody>
                  {#each solicitudesFiltradas as s}
                    <tr>
                      <td><strong>{s.asunto}</strong><small>#{s.id}</small></td>
                      <td>{s.usuario}</td><td>{s.tipo}</td>
                      <td><span class:high={s.prioridad === 'Alta'} class:medium={s.prioridad === 'Media'}>{s.prioridad}</span></td>
                      <td><span class="pill" class:pending={s.estado === 'Pendiente'}
                        class:progress={s.estado === 'En proceso'}
                        class:resolved={s.estado === 'Resuelto'}>{s.estado}</span></td>
                    </tr>
                  {/each}
                </tbody>
              </table>
            </div>
            {#if solicitudesFiltradas.length === 0}<p class="empty">No se encontraron solicitudes.</p>{/if}
          </section>

        {:else}
          <div class="heading">
            <div>
              <span class="eyebrow">HALLAZGOS</span>
              <h1>Objetos perdidos</h1>
              <p class="muted">Registro y seguimiento de objetos encontrados en oficinas y sucursales.</p>
            </div>
            <button class="primary" onclick={() => modalPerdido = true}>+ Registrar objeto</button>
          </div>

          <div class="notice">Evita registrar información personal innecesaria o números completos de tarjetas, documentos y credenciales.</div>

          <section class="lost-grid">
            {#each perdidos as p}
              <article class="panel lost-card">
                <span class="stat-icon purple">⌕</span>
                <h3>{p.nombre}</h3>
                <p class="muted">Lugar: {p.lugar}</p>
                <p class="muted">Fecha: {p.fecha || 'Sin indicar'}</p>
                <span class="pill pending">{p.estado}</span>
              </article>
            {/each}
          </section>
        {/if}

        <footer>Portal conceptual de soporte tecnológico · No es un sistema oficial de APAP.</footer>
      </div>
    </main>

  </div>
{/if}

{#if modalEquipo}

  <div class="overlay">
    <form class="modal" onsubmit={guardarEquipo}>
      <h2>Registrar equipo</h2>
      <p class="muted">Introduce los datos del activo tecnológico.</p>

      <label for="codigo">Código del activo *</label>
      <input id="codigo" bind:value={equipoNuevo.codigo} placeholder="AP-005" required />

      <label for="nombreEquipo">Nombre del equipo *</label>
      <input id="nombreEquipo" bind:value={equipoNuevo.nombre} placeholder="Laptop Dell Latitude" required />

      <label for="usuarioEquipo">Usuario asignado</label>
      <input id="usuarioEquipo" bind:value={equipoNuevo.usuario} placeholder="Nombre del colaborador" />

      <label for="sucursal">Sucursal</label>
      <input id="sucursal" bind:value={equipoNuevo.sucursal} placeholder="Sucursal o área" />

      <label for="condicion">Condición</label>
      <select id="condicion" bind:value={equipoNuevo.condicion}>
        <option>Bueno</option><option>Dañado</option><option>Obsoleto</option>
      </select>

      <label for="estadoEquipo">Estado</label>
      <select id="estadoEquipo" bind:value={equipoNuevo.estado}>
        <option>Disponible</option><option>Asignado</option>
        <option>En reparación</option><option>Evaluar sustitución</option>
        <option>Retirado</option>
      </select>

      <div class="modal-actions">
        <button type="button" class="secondary" onclick={() => modalEquipo = false}>Cancelar</button>
        <button class="primary" type="submit">Guardar equipo</button>
      </div>
    </form>

  </div>
{/if}

{#if modalSolicitud}

  <div class="overlay">
    <form class="modal" onsubmit={guardarSolicitud}>
      <h2>Nueva solicitud técnica</h2>
      <p class="muted">Registra la necesidad del colaborador.</p>

      <label for="asunto">Asunto *</label>
      <input id="asunto" bind:value={solicitudNueva.asunto} placeholder="Equipo no enciende" required />

      <label for="usuarioSolicitud">Nombre del usuario *</label>
      <input id="usuarioSolicitud" bind:value={solicitudNueva.usuario} placeholder="Nombre del colaborador" required />

      <label for="tipoSolicitud">Tipo de solicitud</label>
      <select id="tipoSolicitud" bind:value={solicitudNueva.tipo}>
        <option>Reparación</option><option>Sustitución</option>
        <option>Soporte</option><option>Accesorios</option>
      </select>

      <label for="prioridad">Prioridad</label>
      <select id="prioridad" bind:value={solicitudNueva.prioridad}>
        <option>Baja</option><option>Media</option><option>Alta</option>
      </select>

      <div class="modal-actions">
        <button type="button" class="secondary" onclick={() => modalSolicitud = false}>Cancelar</button>
        <button class="primary" type="submit">Crear solicitud</button>
      </div>
    </form>

  </div>
{/if}

{#if modalPerdido}

  <div class="overlay">
    <form class="modal" onsubmit={guardarPerdido}>
      <h2>Registrar objeto encontrado</h2>
      <p class="muted">Documenta el objeto y su ubicación.</p>

      <label for="objeto">Nombre del objeto *</label>
      <input id="objeto" bind:value={perdidoNuevo.nombre} placeholder="Cargador, adaptador, llaves..." required />

      <label for="lugar">Lugar donde se encontró *</label>
      <input id="lugar" bind:value={perdidoNuevo.lugar} placeholder="Sucursal, piso o área" required />

      <label for="fecha">Fecha del hallazgo *</label>
      <input id="fecha" type="date" bind:value={perdidoNuevo.fecha} required />

      <label for="estadoPerdido">Estado</label>
      <select id="estadoPerdido" bind:value={perdidoNuevo.estado}>
        <option>Pendiente de identificar</option><option>En custodia</option>
        <option>Devuelto</option>
      </select>

      <div class="modal-actions">
        <button type="button" class="secondary" onclick={() => modalPerdido = false}>Cancelar</button>
        <button class="primary" type="submit">Registrar objeto</button>
      </div>
    </form>

  </div>
{/if}

{#if mensaje}

  <div class="toast">{mensaje}</div>
{/if}

<style>
  :global(*) { box-sizing: border-box; }
  :global(body) { margin: 0; font-family: Inter, Arial, sans-serif; background: #f4f6f9; color: #20334a; font-size: 14px; }
  :global(button), :global(input), :global(select) { font: inherit; }
  :global(button) { cursor: pointer; }
  .login { min-height: 100vh; display: grid; grid-template-columns: 1fr 1fr; background: white; }
  .login-brand { padding: 45px 9%; background: linear-gradient(140deg,#0a2243,#174d78); color: white; display: flex; flex-direction: column; justify-content: space-between; }
  .logo { display: flex; align-items: center; gap: 11px; font-size: 21px; font-weight: 800; }
  .logo small { display: block; font-size: 10px; font-weight: 500; margin-top: 3px; opacity: .8; }
  .logo-mark { width: 39px; height: 39px; border-radius: 11px; background: #f58232; color: white; display: grid; place-items: center; font-family: Georgia,serif; font-style: italic; font-size: 30px; }
  .brand-copy { max-width: 470px; margin: 55px 0; }
  .brand-copy h1 { font-size: clamp(32px,4vw,52px); line-height: 1.12; letter-spacing: -1.4px; }
  .brand-copy p { color: #d1deeb; line-height: 1.8; }
  .eyebrow { color: #ef792f; font-size: 10px; font-weight: 800; letter-spacing: 1.5px; }
  .legal { color: #a9bfd5; font-size: 10px; }
  .login-content { display: grid; place-items: center; padding: 35px; }
  .login-form { width: min(100%,390px); }
  h2 { font-size: 29px; margin: 12px 0 7px; }
  .muted { color: #7c8b9d; line-height: 1.6; }
  label { display: block; font-size: 12px; font-weight: 700; margin: 17px 0 8px; }
  input, select { width: 100%; min-width: 0; border: 1px solid #dbe3eb; border-radius: 7px; padding: 12px; background: white; color: #20334a; outline: none; }
  input:focus, select:focus { border-color: #347fa8; box-shadow: 0 0 0 3px #347fa81a; }
  .primary { border: 0; border-radius: 7px; padding: 12px 15px; background: #f58232; color: white; font-weight: 750; }
  .primary:hover { background: #df6d21; }
  .full { width: 100%; margin-top: 23px; }
  .security { margin-top: 25px; padding: 14px; background: #f1f6fa; border-radius: 8px; font-size: 11px; }
  .security p { color: #708196; line-height: 1.6; }
  .error { color: #ba332d; font-size: 12px; }
  .app { min-height: 100vh; display: flex; }
  .sidebar { width: 245px; flex-shrink: 0; padding: 25px 14px; display: flex; flex-direction: column; background: #0b2445; color: white; }
  .sidebar-logo { padding: 0 8px 35px; }
  .nav-label { padding-left: 10px; color: #8ca1b9; font-size: 9px; letter-spacing: 1px; font-weight: 800; }
  nav { display: grid; gap: 6px; }
  .nav-item { display: flex; align-items: center; gap: 12px; text-align: left; border: 0; border-radius: 7px; padding: 13px 12px; color: #c4d2e1; background: transparent; font-size: 12px; }
  .nav-item span { font-size: 18px; width: 20px; }
  .nav-item:hover, .nav-item.active { background: #1a3b60; color: white; }
  .nav-item.active { box-shadow: inset 3px 0 #f58232; }
  .sidebar-bottom { margin-top: auto; }
  .help-card { padding: 14px; margin: 20px 0; border-radius: 9px; background: #153354; font-size: 11px; }
  .help-card p { color: #b2c2d4; line-height: 1.6; }
  .logout { background: transparent; color: #c6d4e3; border: 0; padding: 12px 5px; }
  .main { min-width: 0; flex: 1; }
  .topbar { height: 66px; display: flex; justify-content: space-between; align-items: center; padding: 0 30px; background: white; border-bottom: 1px solid #e6ebf0; color: #8190a1; font-size: 11px; }
  .topbar strong { color: #30445b; }
  .online { color: #728296; }
  .online i { display: inline-block; width: 7px; height: 7px; background: #29a477; border-radius: 50%; margin-right: 6px; }
  .content { max-width: 1450px; margin: auto; padding: 30px; }
  .heading { display: flex; align-items: center; justify-content: space-between; gap: 15px; margin-bottom: 23px; }
  h1 { font-size: 28px; letter-spacing: -.8px; margin: 8px 0; }
  .heading .muted { font-size: 12px; margin: 0; }
  .banner { padding: 25px 28px; color: white; border-radius: 11px; background: linear-gradient(105deg,#0c2b50,#1a5c84); margin-bottom: 19px; }
  .banner h2 { font-size: 26px; margin: 9px 0; }
  .banner p { color: #d5e3ef; font-size: 12px; line-height: 1.6; }
  .stats { display: grid; grid-template-columns: repeat(4,minmax(0,1fr)); gap: 14px; margin-bottom: 19px; }
  .stat { background: white; border: 1px solid #e6ebf0; border-radius: 10px; padding: 17px; min-width: 0; }
  .stat-icon { display: inline-grid; place-items: center; width: 32px; height: 32px; border-radius: 8px; font-size: 17px; font-weight: 800; }
  .blue { background: #eaf2fc; color: #377bb6; }
  .green { background: #e5f7ed; color: #27966a; }
  .orange { background: #fff0e3; color: #e17b31; }
  .purple { background: #f0eaff; color: #8264c8; }
  .stat p { font-size: 11px; color: #738397; }
  .stat strong { display: block; font-size: 31px; margin: 9px 0; }
  .stat small, .stat button { font-size: 10px; color: #7d8c9e; }
  .stat button, .text-button { border: 0; background: transparent; color: #337ea8; padding: 0; font-size: 10px; }
  .panel { background: white; border: 1px solid #e6ebf0; border-radius: 10px; padding: 19px; margin-bottom: 17px; min-width: 0; }
  .panel-heading { display: flex; justify-content: space-between; align-items: center; gap: 10px; margin-bottom: 15px; }
  .panel-heading h3 { font-size: 14px; margin: 0 0 5px; }
  .panel-heading p { font-size: 10px; color: #8795a5; margin: 0; }
  .list-row { display: flex; align-items: center; gap: 12px; padding: 13px 0; border-top: 1px solid #edf0f4; }
  .row-icon { display: grid; place-items: center; width: 34px; height: 34px; background: #edf4fa; color: #397aa6; border-radius: 8px; font-size: 17px; }
  .row-main { flex: 1; min-width: 0; display: grid; gap: 5px; }
  .row-main strong { font-size: 11px; }
  .row-main small { color: #8593a3; font-size: 10px; }
  .pill { display: inline-block; padding: 5px 8px; border-radius: 20px; background: #edf1f5; color: #64758a; font-size: 9px; font-weight: 700; white-space: nowrap; }
  .pending { background: #fff2df; color: #b66d1c; }
  .progress { background: #e9f2ff; color: #326fb2; }
  .resolved { background: #e5f7ed; color: #23845b; }
  .replace { background: #f0eaff; color: #795bb9; }
  .quick-links { display: grid; grid-template-columns: repeat(3,minmax(0,1fr)); gap: 10px; }
  .quick-links button { border: 1px solid #e2e8ef; background: white; padding: 15px 10px; border-radius: 8px; color: #347da6; font-size: 11px; }
  .search { display: flex; align-items: center; gap: 10px; max-width: 440px; padding: 0 12px; height: 40px; margin-bottom: 17px; background: white; border: 1px solid #dfe6ed; border-radius: 7px; color: #8090a1; }
  .search span { font-size: 21px; }
  .search input { border: 0; box-shadow: none; padding: 8px 0; font-size: 11px; }
  .table-panel { padding: 0; overflow: hidden; }
  .table-wrap { overflow-x: auto; }
  table { border-collapse: collapse; width: 100%; text-align: left; white-space: nowrap; }
  th { background: #f8fafc; padding: 14px; font-size: 9px; color: #8795a5; }
  td { padding: 14px; border-top: 1px solid #edf0f4; color: #65758a; font-size: 10px; }
  td strong, td small { display: block; }
  td strong { color: #293e54; font-size: 10px; margin-bottom: 5px; }
  td small { color: #8a97a6; font-size: 9px; }
  .condition { color: #387c5b; font-weight: 700; }
  .table-foot { padding: 13px; color: #8290a0; font-size: 10px; border-top: 1px solid #edf0f4; }
  .notice { padding: 14px; margin: 15px 0; background: #f0f7fc; border: 1px solid #d9eaf5; border-radius: 8px; color: #496d87; font-size: 11px; line-height: 1.7; }
  .high { color: #c24a43; font-weight: 700; }
  .medium { color: #b77a2e; font-weight: 700; }
  .lost-grid { display: grid; grid-template-columns: repeat(2,minmax(0,1fr)); gap: 15px; }
  .lost-card h3 { font-size: 14px; }
  .empty { padding: 22px; text-align: center; color: #8290a0; }
  footer { padding: 18px 0 0; color: #98a4b1; font-size: 9px; }
  .overlay { position: fixed; inset: 0; z-index: 10; background: #07192e99; display: grid; place-items: center; padding: 15px; overflow-y: auto; }
  .modal { width: min(100%, 480px); max-height: calc(100vh - 30px); overflow-y: auto; padding: 24px; background: white; border-radius: 12px; }
  .modal h2 { font-size: 23px; }
  .modal .muted { font-size: 11px; }
  .modal input, .modal select { margin-bottom: 3px; }
  .modal-actions { display: flex; justify-content: flex-end; gap: 9px; border-top: 1px solid #edf0f4; margin-top: 20px; padding-top: 15px; }
  .secondary { border: 1px solid #dce3eb; background: white; color: #586c80; border-radius: 7px; padding: 10px 13px; }
  .toast { position: fixed; right: 20px; bottom: 20px; z-index: 30; max-width: calc(100% - 40px); background: #12365a; color: white; padding: 14px 18px; border-radius: 8px; box-shadow: 0 5px 20px #0002; font-size: 12px; }
  @media (max-width: 1000px) {
    .sidebar { width: 210px; }
    .content { padding: 22px; }
    .stats { grid-template-columns: repeat(2,minmax(0,1fr)); }
  }
  @media (max-width: 700px) {
    .login { grid-template-columns: 1fr; }
    .login-brand { display: none; }
    .login-content { min-height: 100vh; padding: 25px; }
    .app { display: block; }
    .sidebar { width: 100%; min-height: auto; padding: 14px; }
    .sidebar-logo { padding: 0 4px 12px; }
    .nav-label, .sidebar-bottom { display: none; }
    nav { display: flex; overflow-x: auto; }
    .nav-item { white-space: nowrap; padding: 10px; font-size: 10px; }
    .topbar { height: 52px; padding: 0 13px; }
    .online { display: none; }
    .content { padding: 19px 13px; }
    .heading { align-items: flex-start; flex-direction: column; }
    .heading > .primary { width: 100%; }
    h1 { font-size: 25px; }
    .stats { gap: 9px; }
    .stat { padding: 12px; }
    .stat strong { font-size: 27px; }
    .quick-links { grid-template-columns: 1fr; }
    .lost-grid { grid-template-columns: 1fr; }
  }
</style>
