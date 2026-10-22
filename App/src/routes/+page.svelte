<script lang="ts">
  import { onMount } from 'svelte';
  import { createClient } from '@supabase/supabase-js';

  // Configuración de Supabase
  const supabaseUrl = 'https://ysuquxlteguzwrihfgko.supabase.co';
  const supabaseAnonKey = 'sb_publishable_tZsxsrAbXKq5R07IhRP3RA_uvWyxgA-';
  const supabase = createClient(supabaseUrl, supabaseAnonKey);

  type Section = 'inicio' | 'equipos' | 'solicitudes' | 'perdidos';

  type Equipo = {
    id: number;
    codigo: string;
    nombre: string;
    usuario: string;
    sucursal: string;
    condicion: string;
    estado: string;
  };

  type Solicitud = {
    id: number;
    asunto: string;
    usuario: string;
    tipo: string;
    prioridad: string;
    estado: string;
  };

  type ObjetoPerdido = {
    id?: number;
    nombre: string;
    lugar: string;
    fecha: string;
    estado: string;
  };

  // Estados reactivos (Svelte 5 Runes)
  let seccion = $state<Section>('inicio');
  let sesion = $state(false);
  let correo = $state('');
  let contrasena = $state('');
  let error = $state('');
  let busqueda = $state('');
  let modalEquipo = $state(false);
  let modalSolicitud = $state(false);
  let modalPerdido = $state(false);
  let mensaje = $state('');

  let equipos = $state<Equipo[]>([]);
  let solicitudes = $state<Solicitud[]>([]);
  let perdidos = $state<ObjetoPerdido[]>([]);

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

  // Carga de datos desde Supabase
  async function cargarDatos() {
    const { data: eq } = await supabase.from('equipos').select('*').order('id', { ascending: false });
    if (eq) equipos = eq;

    const { data: sol } = await supabase.from('solicitudes').select('*').order('id', { ascending: false });
    if (sol) solicitudes = sol;

    const { data: per } = await supabase.from('objetos_perdidos').select('*').order('id', { ascending: false });
    if (per) perdidos = per;
  }

  // Valores derivados
  let equiposFiltrados = $derived(
    equipos.filter(e =>
      Object.values(e).join(' ').toLowerCase().includes(busqueda.toLowerCase())
    )
  );

  let solicitudesFiltradas = $derived(
    solicitudes.filter(s =>
      Object.values(s).join(' ').toLowerCase().includes(busqueda.toLowerCase())
    )
  );

  let totalBuenos = $derived(equipos.filter(e => e.condicion === 'Bueno').length);
  let totalReparacion = $derived(equipos.filter(e => e.estado === 'En reparación').length);
  let totalSustitucion = $derived(equipos.filter(e => e.estado === 'Evaluar sustitución').length);

  function notificar(texto: string) {
    mensaje = texto;
    setTimeout(() => mensaje = '', 3000);
  }

  // Validación de inicio de sesión
  async function iniciarSesion(event: SubmitEvent) {
    event.preventDefault();
    error = '';

    const correoLimpio = correo.trim();
    const passLimpia = contrasena.trim();

    if (!correoLimpio.includes('@')) {
      error = 'Introduce un correo válido.';
      return;
    }

    // Consulta flexible ignorando mayúsculas/minúsculas en el correo
    const { data: usuarioEncontrado, error: dbError } = await supabase
      .from('usuarios_portal')
      .select('*')
      .ilike('correo', correoLimpio)
      .maybeSingle();

    if (dbError) {
      error = 'Error de conexión con la base de datos.';
      return;
    }

    if (!usuarioEncontrado) {
      error = 'Correo o contraseña incorrectos.';
      return;
    }

    // Validación estricta de contraseña
    if (usuarioEncontrado.contrasena !== passLimpia) {
      error = 'Correo o contraseña incorrectos.';
      return;
    }

    sesion = true;
    await cargarDatos();
    notificar('Bienvenido/a, ' + (usuarioEncontrado.nombre_completo || 'Colaborador'));
  }

  // Funciones para guardar nuevos registros en Supabase
  async function guardarEquipo(event: SubmitEvent) {
    event.preventDefault();
    const { data, error: dbError } = await supabase.from('equipos').insert([equipoNuevo]).select();
    if (dbError) { notificar('Error: ' + dbError.message); return; }
    if (data) equipos = [...data, ...equipos];
    equipoNuevo = { codigo: '', nombre: '', usuario: '', sucursal: '', condicion: 'Bueno', estado: 'Disponible' };
    modalEquipo = false;
    notificar('Equipo registrado.');
  }

  async function guardarSolicitud(event: SubmitEvent) {
    event.preventDefault();
    const { data, error: dbError } = await supabase.from('solicitudes').insert([{ ...solicitudNueva, estado: 'Pendiente' }]).select();
    if (dbError) { notificar('Error: ' + dbError.message); return; }
    if (data) solicitudes = [...data, ...solicitudes];
    solicitudNueva = { asunto: '', usuario: '', tipo: 'Reparación', prioridad: 'Media' };
    modalSolicitud = false;
    notificar('Solicitud creada.');
  }

  async function guardarPerdido(event: SubmitEvent) {
    event.preventDefault();
    const { data, error: dbError } = await supabase.from('objetos_perdidos').insert([perdidoNuevo]).select();
    if (dbError) { notificar('Error: ' + dbError.message); return; }
    if (data) perdidos = [...data, ...perdidos];
    perdidoNuevo = { nombre: '', lugar: '', fecha: '', estado: 'Pendiente de identificar' };
    modalPerdido = false;
    notificar('Objeto registrado.');
  }

  function abrir(seccionNueva: Section) {
    seccion = seccionNueva;
    busqueda = '';
  }
</script>

<svelte:head>
  <title>Centro de Soporte | Portal interno</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
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
        <p>Gestiona equipos, solicitudes y objetos encontrados desde un solo lugar.</p>
      </div>
      <small class="legal">Seguridad verificada con base de datos</small>
    </section>

    <section class="login-content">
      <form class="login-form" onsubmit={iniciarSesion}>
        <span class="eyebrow">ACCESO RESTRINGIDO</span>
        <h2>Iniciar sesión</h2>
        <p class="muted">Introduce tus credenciales autorizadas.</p>

        <label for="correo">Correo institucional</label>
        <input id="correo" type="email" bind:value={correo} placeholder="ejemplo@gmail.com" required />

        <label for="contrasena">Contraseña</label>
        <input id="contrasena" type="password" bind:value={contrasena} placeholder="" required />

        {#if error}
          <p class="error">{error}</p>
        {/if}

        <button class="primary full" type="submit">Entrar al portal →</button>
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
          <button class:active={seccion === item.id} class="nav-item" onclick={() => abrir(item.id as Section)}>
            <span>{item.icono}</span>
            <span>{item.nombre}</span>
          </button>
        {/each}
      </nav>
      <div class="sidebar-bottom">
        <button class="logout" onclick={() => sesion = false}>↪ Cerrar sesión</button>
      </div>
    </aside>

    <main class="main">
      <header class="topbar">
        <span>Portal interno / <strong>{menu.find(m => m.id === seccion)?.nombre}</strong></span>
        <span class="online"><i></i> Conectado y verificado</span>
      </header>

      <div class="content">
        {#if seccion === 'inicio'}
          <div class="heading">
            <div>
              <span class="eyebrow">CENTRO DE CONTROL</span>
              <h1>Resumen general</h1>
              <p class="muted">Datos en tiempo real desde Supabase.</p>
            </div>
            <button class="primary action-btn" onclick={() => modalSolicitud = true}>+ Nueva solicitud</button>
          </div>

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
            </article>
            <article class="stat">
              <span class="stat-icon orange">↻</span>
              <p>En reparación</p>
              <strong>{totalReparacion}</strong>
            </article>
            <article class="stat">
              <span class="stat-icon purple">⇄</span>
              <p>Por sustituir</p>
              <strong>{totalSustitucion}</strong>
            </article>
          </section>

          <section class="panel">
            <div class="panel-heading">
              <div><h3>Solicitudes recientes</h3></div>
              <button class="text-button" onclick={() => abrir('solicitudes')}>Ver todas →</button>
            </div>
            {#each solicitudes.slice(0, 4) as s}
              <div class="list-row">
                <div class="row-icon">▤</div>
                <div class="row-main">
                  <strong>{s.asunto}</strong>
                  <small>#{s.id} · {s.usuario} · {s.tipo}</small>
                </div>
                <span class="pill">{s.estado}</span>
              </div>
            {/each}
          </section>

        {:else if seccion === 'equipos'}
          <div class="heading">
            <h1>Inventario de equipos</h1>
            <button class="primary action-btn" onclick={() => modalEquipo = true}>+ Registrar equipo</button>
          </div>
          <div class="search">
            <span>⌕</span>
            <input bind:value={busqueda} placeholder="Buscar equipo..." />
          </div>
          <section class="panel table-panel">
            <div class="table-wrap">
              <table>
                <thead>
                  <tr><th>CÓDIGO / EQUIPO</th><th>USUARIO</th><th>SUCURSAL</th><th>CONDICIÓN</th><th>ESTADO</th></tr>
                </thead>
                <tbody>
                  {#each equiposFiltrados as e}
                    <tr>
                      <td><strong>{e.nombre}</strong><small>{e.codigo}</small></td>
                      <td>{e.usuario}</td>
                      <td>{e.sucursal}</td>
                      <td>{e.condicion}</td>
                      <td><span class="pill">{e.estado}</span></td>
                    </tr>
                  {/each}
                </tbody>
              </table>
            </div>
          </section>

        {:else if seccion === 'solicitudes'}
          <div class="heading">
            <h1>Solicitudes técnicas</h1>
            <button class="primary action-btn" onclick={() => modalSolicitud = true}>+ Nueva solicitud</button>
          </div>
          <section class="panel table-panel">
            <div class="table-wrap">
              <table>
                <thead>
                  <tr><th>ASUNTO</th><th>USUARIO</th><th>TIPO</th><th>PRIORIDAD</th><th>ESTADO</th></tr>
                </thead>
                <tbody>
                  {#each solicitudesFiltradas as s}
                    <tr>
                      <td><strong>{s.asunto}</strong></td>
                      <td>{s.usuario}</td>
                      <td>{s.tipo}</td>
                      <td>{s.prioridad}</td>
                      <td><span class="pill">{s.estado}</span></td>
                    </tr>
                  {/each}
                </tbody>
              </table>
            </div>
          </section>

        {:else}
          <div class="heading">
            <h1>Objetos perdidos</h1>
            <button class="primary action-btn" onclick={() => modalPerdido = true}>+ Registrar objeto</button>
          </div>
          <section class="lost-grid">
            {#each perdidos as p}
              <article class="panel lost-card">
                <h3>{p.nombre}</h3>
                <p class="muted">Lugar: {p.lugar}</p>
                <p class="muted">Fecha: {p.fecha}</p>
                <span class="pill">{p.estado}</span>
              </article>
            {/each}
          </section>
        {/if}
      </div>
    </main>
  </div>
{/if}

<!-- MODALES DE REGISTRO -->
{#if modalEquipo}
  <div class="overlay">
    <form class="modal" onsubmit={guardarEquipo}>
      <h2>Registrar equipo</h2>
      <label>Código *</label>
      <input bind:value={equipoNuevo.codigo} required />
      <label>Nombre *</label>
      <input bind:value={equipoNuevo.nombre} required />
      <label>Usuario</label>
      <input bind:value={equipoNuevo.usuario} />
      <label>Sucursal</label>
      <input bind:value={equipoNuevo.sucursal} />
      <div class="modal-actions">
        <button type="button" class="secondary" onclick={() => modalEquipo = false}>Cancelar</button>
        <button class="primary" type="submit">Guardar</button>
      </div>
    </form>
  </div>
{/if}

{#if modalSolicitud}
  <div class="overlay">
    <form class="modal" onsubmit={guardarSolicitud}>
      <h2>Nueva solicitud</h2>
      <label>Asunto *</label>
      <input bind:value={solicitudNueva.asunto} required />
      <label>Usuario *</label>
      <input bind:value={solicitudNueva.usuario} required />
      <div class="modal-actions">
        <button type="button" class="secondary" onclick={() => modalSolicitud = false}>Cancelar</button>
        <button class="primary" type="submit">Crear</button>
      </div>
    </form>
  </div>
{/if}

{#if modalPerdido}
  <div class="overlay">
    <form class="modal" onsubmit={guardarPerdido}>
      <h2>Registrar objeto</h2>
      <label>Nombre *</label>
      <input bind:value={perdidoNuevo.nombre} required />
      <label>Lugar *</label>
      <input bind:value={perdidoNuevo.lugar} required />
      <label>Fecha *</label>
      <input type="date" bind:value={perdidoNuevo.fecha} required />
      <div class="modal-actions">
        <button type="button" class="secondary" onclick={() => modalPerdido = false}>Cancelar</button>
        <button class="primary" type="submit">Guardar</button>
      </div>
    </form>
  </div>
{/if}

{#if mensaje}
  <div class="toast">{mensaje}</div>
{/if}

<style>
  :global(*) { box-sizing: border-box; }
  :global(body) { margin: 0; font-family: Inter, system-ui, -apple-system, sans-serif; background: #f8fafc; color: #0f172a; font-size: 14px; }
  
  .login { min-height: 100vh; display: grid; grid-template-columns: 1fr 1fr; background: #fff; }
  .login-brand { padding: 40px; background: linear-gradient(135deg, #0f2b5c, #1e40af); color: #fff; display: flex; flex-direction: column; justify-content: space-between; }
  .logo { display: flex; align-items: center; gap: 10px; font-size: 20px; font-weight: 800; }
  .logo-mark { width: 35px; height: 35px; background: #2563eb; color: #fff; display: grid; place-items: center; border-radius: 6px; font-weight: bold; }
  .login-content { display: grid; place-items: center; padding: 30px; }
  .login-form { width: 100%; max-width: 380px; }
  .eyebrow { font-size: 11px; font-weight: 800; color: #2563eb; letter-spacing: 0.5px; }
  .login-brand .eyebrow { color: #93c5fd; }
  
  label { display: block; font-size: 12px; font-weight: 700; margin: 15px 0 5px; }
  input, select { width: 100%; border: 1px solid #cbd5e1; border-radius: 6px; padding: 10px; outline: none; }
  input:focus { border-color: #2563eb; }
  
  .primary { border: 0; border-radius: 6px; padding: 10px 15px; background: #2563eb; color: #fff; font-weight: 700; cursor: pointer; transition: background 0.2s; }
  .primary:hover { background: #1d4ed8; }
  .full { width: 100%; margin-top: 20px; }
  .error { color: #dc2626; font-size: 12px; font-weight: bold; margin-top: 10px; }
  
  .app { min-height: 100vh; display: flex; }
  .sidebar { width: 240px; background: #0f172a; color: #fff; padding: 20px; display: flex; flex-direction: column; }
  .nav-label { font-size: 10px; font-weight: 800; color: #64748b; margin: 20px 0 10px; }
  .nav-item { display: flex; gap: 10px; width: 100%; padding: 10px; background: transparent; border: 0; color: #94a3b8; border-radius: 6px; text-align: left; cursor: pointer; margin-bottom: 5px; }
  .nav-item.active, .nav-item:hover { background: #1e293b; color: #fff; }
  .sidebar-bottom { margin-top: auto; }
  .logout { background: transparent; color: #94a3b8; border: 0; cursor: pointer; padding: 10px 0; }
  
  .main { flex: 1; display: flex; flex-direction: column; }
  .topbar { height: 60px; background: #fff; border-bottom: 1px solid #e2e8f0; display: flex; justify-content: space-between; align-items: center; padding: 0 25px; }
  .online i { display: inline-block; width: 8px; height: 8px; background: #16a34a; border-radius: 50%; margin-right: 5px; }
  .content { padding: 25px; max-width: 1400px; margin: 0 auto; width: 100%; }
  
  .heading { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
  .stats { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin-bottom: 20px; }
  .stat { background: #fff; border: 1px solid #e2e8f0; padding: 15px; border-radius: 8px; }
  .stat-icon { width: 30px; height: 30px; display: grid; place-items: center; border-radius: 6px; font-weight: bold; margin-bottom: 8px; }
  .stat button { background: transparent; border: 0; color: #2563eb; padding: 0; margin-top: 10px; cursor: pointer; font-size: 12px; font-weight: 600; }
  
  .blue { background: #eff6ff; color: #2563eb; }
  .green { background: #f0fdf4; color: #16a34a; }
  .orange { background: #fefce8; color: #ca8a04; }
  .purple { background: #f5f3ff; color: #7c3aed; }
  
  .panel { background: #fff; border: 1px solid #e2e8f0; border-radius: 8px; padding: 20px; margin-bottom: 20px; }
  .panel-heading { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; }
  .text-button { background: transparent; border: 0; color: #2563eb; cursor: pointer; font-weight: 600; }
  
  .list-row { display: flex; align-items: center; gap: 12px; padding: 10px 0; border-top: 1px solid #f1f5f9; }
  .row-icon { width: 30px; height: 30px; background: #eff6ff; color: #2563eb; display: grid; place-items: center; border-radius: 6px; }
  .row-main { flex: 1; display: flex; flex-direction: column; }
  .pill { background: #f1f5f9; padding: 3px 8px; border-radius: 10px; font-size: 10px; font-weight: bold; color: #475569; }
  
  .search { display: flex; align-items: center; gap: 10px; background: #fff; border: 1px solid #cbd5e1; padding: 0 10px; border-radius: 6px; margin-bottom: 15px; max-width: 400px; height: 38px; }
  .search input { border: 0; outline: none; width: 100%; padding: 0; }
  
  .table-panel { padding: 0; overflow: hidden; }
  .table-wrap { overflow-x: auto; }
  table { width: 100%; border-collapse: collapse; text-align: left; }
  th { background: #f8fafc; padding: 12px; font-size: 10px; color: #64748b; border-bottom: 1px solid #e2e8f0; }
  td { padding: 12px; border-top: 1px solid #f1f5f9; }
  td small { display: block; color: #64748b; }
  
  .lost-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 15px; }
  .lost-card { background: #fff; border: 1px solid #e2e8f0; padding: 15px; border-radius: 8px; }
  
  .overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.5); display: grid; place-items: center; z-index: 100; padding: 15px; }
  .modal { background: #fff; width: 100%; max-width: 450px; padding: 25px; border-radius: 8px; border-top: 4px solid #2563eb; max-height: 90vh; overflow-y: auto; }
  .modal-actions { display: flex; justify-content: flex-end; gap: 10px; margin-top: 20px; }
  .secondary { background: #fff; border: 1px solid #cbd5e1; padding: 8px 12px; border-radius: 6px; cursor: pointer; }
  
  .toast { position: fixed; right: 20px; bottom: 20px; background: #0f172a; color: #fff; padding: 12px 18px; border-radius: 6px; z-index: 200; font-size: 12px; }

  @media (max-width: 768px) {
    .login { grid-template-columns: 1fr; }
    .login-brand { display: none; }
    .app { flex-direction: column; }
    .sidebar { width: 100%; }
  }
</style>