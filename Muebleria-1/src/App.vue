<template>
  <div class="app">

    <!-- LOGIN -->
    <div v-if="!usuario" class="login-screen">
      <div class="login-bg"></div>
      <div class="login-card">
        <div class="login-brand">
          <div class="brand-logo">AC</div>
          <div>
            <div class="login-title">Arboleda Casa</div>
            <div class="login-sub">MUEBLERÍA PREMIUM · CDMX</div>
          </div>
        </div>
        <h2 class="login-heading">Iniciar sesión</h2>
        <p class="login-desc">Accede a tu cuenta para explorar nuestra colección</p>
        <div class="login-form">
          <div class="lgroup">
            <label>Correo electrónico</label>
            <input v-model="loginForm.email" type="email" placeholder="correo@ejemplo.com" @keyup.enter="login">
          </div>
          <div class="lgroup">
            <label>Contraseña</label>
            <div class="pass-wrap">
              <input v-model="loginForm.password" :type="verPass?'text':'password'" placeholder="••••••••" @keyup.enter="login">
              <button class="pass-toggle" @click="verPass=!verPass">{{ verPass?'🙈':'👁️' }}</button>
            </div>
          </div>
          <div v-if="loginError" class="login-error">{{ loginError }}</div>
          <button class="login-btn" @click="login" :disabled="loginCargando">
            {{ loginCargando ? 'Verificando...' : 'Entrar →' }}
          </button>
        </div>
        <div class="login-hints">
          <p style="font-size:11px;color:#aaa;margin-bottom:8px;font-weight:600;text-transform:uppercase;letter-spacing:0.5px">Cuentas de prueba</p>
          <div class="hint" @click="autoFill('admin')">
            <span class="hint-rol">Admin</span>
            <span>g.montoya@arboledacasa.mx</span>
          </div>
          <div class="hint" @click="autoFill('cliente')">
            <span class="hint-rol cliente">Cliente</span>
            <span>cvenegas@gmail.com</span>
          </div>
        </div>
      </div>
    </div>

    <!-- APP PRINCIPAL -->
    <template v-else>

      <!-- NAVBAR (componente) -->
      <NavBar
        :usuario="usuario"
        :categorias="categorias"
        :filtro-categoria="filtroCategoria"
        :total-carrito="totalCarrito"
        :es-admin="esAdmin"
        v-model:busqueda="busqueda"
        @abrir-carrito="modalActivo='carrito'"
        @set-categoria="setCategoria"
        @nueva-pieza="abrirAdmin(null)"
        @ver-pedidos="verPedidos"
        @cerrar-sesion="cerrarSesion"
      />

      <!-- HERO -->
      <div class="hero">
        <transition name="fade">
          <div class="hero-slide" :key="slideActual"
            :style="{ backgroundImage: `url(${slides[slideActual].img})` }">
            <div class="hero-content">
              <span class="hero-badge">{{ slides[slideActual].badge }}</span>
              <h1 class="hero-title">{{ slides[slideActual].titulo }}</h1>
              <p class="hero-desc">{{ slides[slideActual].desc }}</p>
              <button class="hero-btn" @click="setCategoria(slides[slideActual].cat)">Explorar colección →</button>
            </div>
          </div>
        </transition>
        <button class="arrow left" @click="prevSlide">‹</button>
        <button class="arrow right" @click="nextSlide">›</button>
        <div class="dots">
          <span v-for="(s,i) in slides" :key="i" :class="['dot',i===slideActual?'active':'']" @click="slideActual=i"></span>
        </div>
      </div>

      <!-- STATS -->
      <div class="stats">
        <div class="stat" v-for="s in stats" :key="s.label">
          <div class="stat-val">{{ s.val }}</div>
          <div class="stat-label">{{ s.label }}</div>
        </div>
      </div>

      <!-- CATÁLOGO -->
      <div class="catalogo" ref="catalogo">
        <div class="cat-header">
          <span class="cat-count">{{ mueblesFiltrados.length }} piezas · {{ filtroCategoria || 'todas las categorías' }}</span>
        </div>
        <div class="grid" v-if="mueblesFiltrados.length">
          <!-- ProductCard (componente) -->
          <ProductCard
            v-for="m in mueblesFiltrados"
            :key="m.id"
            :mueble="m"
            :es-admin="esAdmin"
            @ver-detalle="verDetalle"
            @agregar-carrito="agregarCarrito"
            @editar="abrirAdmin"
            @pedir-eliminar="confirmarEliminar"
          />
        </div>
        <div v-else class="empty">No se encontraron muebles</div>
      </div>

      <!-- MODAL DETALLE -->
      <div v-if="modalActivo==='detalle' && muebleDetalle" class="overlay" @click.self="modalActivo=null">
        <div class="modal-detalle">
          <div class="modal-breadcrumb">Catálogo › {{ muebleDetalle.categoria }}</div>
          <div class="modal-header">
            <h2>{{ muebleDetalle.nombre }}</h2>
            <button class="modal-close" @click="modalActivo=null">✕</button>
          </div>
          <div class="modal-body">
            <img :src="getImg(muebleDetalle)" :alt="muebleDetalle.nombre" class="modal-img" @error="onImgError">
            <div class="modal-info">
              <div class="modal-brand">✦ ARBOLEDA CASA</div>
              <p class="modal-desc">{{ muebleDetalle.descripcion }}</p>
              <div class="modal-stats">
                <div class="mstat"><div class="mstat-label">PRECIO</div><div class="mstat-val">${{ muebleDetalle.precio.toLocaleString() }}</div></div>
                <div class="mstat"><div class="mstat-label">STOCK</div><div class="mstat-val">{{ muebleDetalle.stock }} uds.</div></div>
                <div class="mstat"><div class="mstat-label">CATEGORÍA</div><div class="mstat-val">{{ muebleDetalle.categoria }}</div></div>
              </div>
            </div>
          </div>
          <div class="modal-actions">
            <button class="btn-cerrar" @click="modalActivo=null">Cerrar</button>
            <button class="btn-agregar" @click="agregarCarrito(muebleDetalle); modalActivo=null" :disabled="muebleDetalle.stock===0">
              {{ muebleDetalle.stock===0 ? 'Agotado' : '🛒 Agregar al carrito' }}
            </button>
          </div>
        </div>
      </div>

      <!-- MODAL CARRITO (componente) -->
      <CartModal
        v-if="modalActivo==='carrito'"
        :carrito="carrito"
        :total-precio="totalPrecio"
        @cerrar="modalActivo=null"
        @cambiar-qty="cambiarQty"
        @quitar="quitarCarrito"
        @iniciar-compra="iniciarCompra"
      />

      <!-- MODAL CHECKOUT -->
      <div v-if="modalActivo==='checkout'" class="overlay" @click.self="modalActivo=null">
        <div class="modal-checkout">
          <div class="modal-header">
            <h2>💳 Finalizar compra</h2>
            <button class="modal-close" @click="modalActivo=null">✕</button>
          </div>
          <div v-if="pasoCheckout===1" class="checkout-body">
            <div class="checkout-section">
              <h3 class="checkout-section-title">📦 Datos de envío</h3>
              <div class="checkout-grid">
                <div class="fgroup full">
                  <label>Nombre completo</label>
                  <input v-model="checkout.nombre" :placeholder="usuario.nombre">
                </div>
                <div class="fgroup full">
                  <label>Dirección</label>
                  <input v-model="checkout.direccion" placeholder="Calle, número, colonia">
                </div>
                <div class="fgroup">
                  <label>Ciudad</label>
                  <input v-model="checkout.ciudad" placeholder="Ciudad de México">
                </div>
                <div class="fgroup">
                  <label>Código postal</label>
                  <input v-model="checkout.cp" placeholder="06600">
                </div>
                <div class="fgroup full">
                  <label>Teléfono</label>
                  <input v-model="checkout.telefono" placeholder="55 1234 5678">
                </div>
              </div>
            </div>
            <div class="checkout-resumen">
              <div class="resumen-item" v-for="item in carrito" :key="item.id">
                <span>{{ item.nombre }} x{{ item.qty }}</span>
                <span>${{ (item.precio*item.qty).toLocaleString() }}</span>
              </div>
              <div class="resumen-total">
                <span>Total</span>
                <span>${{ totalPrecio.toLocaleString() }}</span>
              </div>
            </div>
            <div class="checkout-actions">
              <button class="btn-cerrar" @click="modalActivo='carrito'">← Regresar</button>
              <button class="btn-agregar" @click="irPaso2">Continuar al pago →</button>
            </div>
          </div>
          <div v-if="pasoCheckout===2" class="checkout-body">
            <div class="checkout-section">
              <h3 class="checkout-section-title">💳 Método de pago</h3>
              <div class="metodos-pago">
                <div :class="['metodo', checkout.metodo==='tarjeta'?'active':'']" @click="checkout.metodo='tarjeta'">
                  <span>💳</span> Tarjeta de crédito/débito
                </div>
                <div :class="['metodo', checkout.metodo==='transferencia'?'active':'']" @click="checkout.metodo='transferencia'">
                  <span>🏦</span> Transferencia bancaria
                </div>
                <div :class="['metodo', checkout.metodo==='efectivo'?'active':'']" @click="checkout.metodo='efectivo'">
                  <span>💵</span> Pago en efectivo (OXXO)
                </div>
              </div>
              <div v-if="checkout.metodo==='tarjeta'" class="tarjeta-form">
                <div class="fgroup full">
                  <label>Número de tarjeta</label>
                  <input v-model="checkout.tarjeta" placeholder="1234 5678 9012 3456" maxlength="19">
                </div>
                <div class="fgroup">
                  <label>Vencimiento</label>
                  <input v-model="checkout.vencimiento" placeholder="MM/AA" maxlength="5">
                </div>
                <div class="fgroup">
                  <label>CVV</label>
                  <input v-model="checkout.cvv" placeholder="123" maxlength="3" type="password">
                </div>
              </div>
              <div v-if="checkout.metodo==='transferencia'" class="info-pago">
                <p>Banco: BBVA · Cuenta: 1234 5678 9012</p>
                <p>CLABE: 012 180 00123456789 0</p>
                <p>Beneficiario: Arboleda Casa S.A. de C.V.</p>
              </div>
              <div v-if="checkout.metodo==='efectivo'" class="info-pago">
                <p>Al confirmar recibirás una referencia de pago para OXXO.</p>
                <p>Tienes 48 horas para realizar el pago.</p>
              </div>
            </div>
            <div class="resumen-total" style="margin:0 24px 16px">
              <span>Total a pagar</span>
              <span style="font-size:22px;font-weight:700;color:#2d6a4f">${{ totalPrecio.toLocaleString() }}</span>
            </div>
            <div class="checkout-actions">
              <button class="btn-cerrar" @click="pasoCheckout=1">← Regresar</button>
              <button class="btn-comprar" @click="confirmarCompra" :disabled="comprando">
                {{ comprando ? 'Procesando...' : '✓ Confirmar compra' }}
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- MODAL PEDIDO EXITOSO -->
      <div v-if="modalActivo==='exito'" class="overlay">
        <div class="modal-exito">
          <div class="exito-icon">✓</div>
          <h2 class="exito-title">¡Compra realizada!</h2>
          <p class="exito-desc">Tu pedido #{{ ultimoPedido }} ha sido confirmado. Te llegará un correo con los detalles del envío.</p>
          <div class="exito-detalle">
            <div class="exito-row" v-for="item in ultimosItems" :key="item.id">
              <span>{{ item.nombre }} x{{ item.qty }}</span>
              <span>${{ (item.precio*item.qty).toLocaleString() }}</span>
            </div>
            <div class="exito-total">
              <span>Total pagado</span>
              <span>${{ ultimoTotal.toLocaleString() }}</span>
            </div>
          </div>
          <button class="btn-agregar" style="width:100%;margin-top:8px" @click="modalActivo=null">Seguir comprando</button>
        </div>
      </div>

      <!-- MODAL PEDIDOS -->
      <div v-if="modalActivo==='pedidos'" class="overlay" @click.self="modalActivo=null">
        <div class="modal-pedidos">
          <div class="modal-header">
            <h2>📦 Mis pedidos</h2>
            <button class="modal-close" @click="modalActivo=null">✕</button>
          </div>
          <div v-if="misPedidos.length" class="pedidos-lista">
            <div class="pedido-card" v-for="p in misPedidos" :key="p.id">
              <div class="pedido-header">
                <span class="pedido-id">Pedido #{{ p.id }}</span>
                <span class="pedido-fecha">{{ p.fecha }}</span>
                <span class="pedido-status">{{ p.estado }}</span>
              </div>
              <div class="pedido-items">
                <span v-for="item in p.items" :key="item.id">{{ item.nombre }} x{{ item.qty }}</span>
              </div>
              <div class="pedido-total">Total: ${{ p.total.toLocaleString() }}</div>
            </div>
          </div>
          <div v-else class="empty" style="padding:40px">No tienes pedidos aún</div>
        </div>
      </div>

      <!-- ADMIN FORM (componente) -->
      <AdminForm
        v-if="modalActivo==='admin' && esAdmin"
        :editando-id="editandoId"
        :categorias="categorias"
        :form-inicial="form"
        @guardar="guardar"
        @cancelar="cerrarAdmin"
      />

      <!-- MODAL CONFIRMAR ELIMINAR -->
      <div v-if="modalActivo==='confirmar' && confirmItem" class="overlay" @click.self="modalActivo=null">
        <div class="modal-confirm">
          <h3>Eliminar mueble</h3>
          <p>¿Eliminar "{{ confirmItem.nombre }}"? Esta acción no se puede deshacer.</p>
          <div class="modal-actions">
            <button class="btn-cerrar" @click="modalActivo=null">Cancelar</button>
            <button class="btn-danger" @click="eliminar(confirmItem.id)">Eliminar</button>
          </div>
        </div>
      </div>

    </template>

    <div :class="['toast', toastVisible?'show':'']">{{ toastMsg }}</div>
  </div>
</template>

<script>
import NavBar from './components/NavBar.vue'
import ProductCard from './components/ProductCard.vue'
import CartModal from './components/CartModal.vue'
import AdminForm from './components/AdminForm.vue'

const API = 'http://localhost:3001'
const IMGS = {
  'Sala':'https://images.unsplash.com/photo-1555041469-a586c61ea9bc?w=600&q=80',
  'Recámara':'https://images.unsplash.com/photo-1540518614846-7eded433c457?w=600&q=80',
  'Comedor':'https://images.unsplash.com/photo-1617806118233-18e1de247200?w=600&q=80',
  'Oficina':'https://images.unsplash.com/photo-1593642632559-0c6d3fc62b89?w=600&q=80',
}

export default {
  components: { NavBar, ProductCard, CartModal, AdminForm },
  data() {
    return {
      usuario:null, verPass:false,
      loginForm:{email:'',password:''}, loginError:'', loginCargando:false,
      muebles:[], carrito:[],
      busqueda:'', filtroCategoria:'',
      categorias:['Todas','Sala','Recámara','Comedor','Oficina'],
      slideActual:0,
      slides:[
        {badge:'NUEVA COLECCIÓN',titulo:'Espacios que inspiran',desc:'Diseño mexicano con materiales nobles y acabados premium',cat:'Sala',img:'https://images.unsplash.com/photo-1555041469-a586c61ea9bc?w=1400&q=80'},
        {badge:'RECÁMARAS',titulo:'Tu descanso, nuestro arte',desc:'Camas y accesorios diseñados para el descanso perfecto',cat:'Recámara',img:'https://images.unsplash.com/photo-1540518614846-7eded433c457?w=1400&q=80'},
        {badge:'HOME OFFICE',titulo:'Trabaja con estilo',desc:'Escritorios y sillas ergonómicas para tu oficina en casa',cat:'Oficina',img:'https://images.unsplash.com/photo-1593642632559-0c6d3fc62b89?w=1400&q=80'},
      ],
      stats:[],
      modalActivo:null, muebleDetalle:null, confirmItem:null,
      editandoId:null,
      form:{nombre:'',categoria:'Sala',precio:0,stock:0,descripcion:'',imagen:''},
      pasoCheckout:1, comprando:false,
      checkout:{nombre:'',direccion:'',ciudad:'',cp:'',telefono:'',metodo:'tarjeta',tarjeta:'',vencimiento:'',cvv:''},
      misPedidos:[],
      ultimoPedido:null, ultimoTotal:0, ultimosItems:[],
      toastMsg:'', toastVisible:false,
      slideInterval:null,
    }
  },
  computed: {
    esAdmin(){ return this.usuario?.rol==='admin' },
    mueblesFiltrados(){
      return this.muebles.filter(m=>
        (!this.busqueda||m.nombre.toLowerCase().includes(this.busqueda.toLowerCase())||m.descripcion.toLowerCase().includes(this.busqueda.toLowerCase()))&&
        (!this.filtroCategoria||m.categoria===this.filtroCategoria)
      )
    },
    totalCarrito(){ return this.carrito.reduce((a,c)=>a+c.qty,0) },
    totalPrecio(){ return this.carrito.reduce((a,c)=>a+c.precio*c.qty,0) }
  },
  beforeUnmount() {
    clearInterval(this.slideInterval)
  },
  methods: {
    autoFill(rol){
      if(rol==='admin'){ this.loginForm={email:'g.montoya@arboledacasa.mx',password:'Arboleda2026#'} }
      else { this.loginForm={email:'cvenegas@gmail.com',password:'Carlos2026!'} }
    },
    async login(){
      this.loginError=''
      if(!this.loginForm.email||!this.loginForm.password){ this.loginError='Completa todos los campos'; return }
      this.loginCargando=true
      try{
        const r=await fetch(`${API}/usuarios?email=${encodeURIComponent(this.loginForm.email)}&password=${encodeURIComponent(this.loginForm.password)}`)
        const users=await r.json()
        if(users.length>0){
          this.usuario=users[0]
          this.loginForm={email:'',password:''}
          await this.cargar()
          await this.cargarPedidos()
          this.slideInterval=setInterval(this.nextSlide,4500)
          this.toast(`Bienvenida/o, ${this.usuario.nombre.split(' ')[0]} 👋`)
        } else { this.loginError='Correo o contraseña incorrectos' }
      } catch(e){ this.loginError='⚠️ No se pudo conectar al servidor' }
      this.loginCargando=false
    },
    cerrarSesion(){
      clearInterval(this.slideInterval)
      this.usuario=null; this.carrito=[]; this.modalActivo=null
    },
    async cargar(){
      try{ const r=await fetch(`${API}/muebles`); this.muebles=await r.json() }
      catch(e){ this.toast('⚠️ Error al cargar muebles') }
    },
    async cargarPedidos(){
      try{
        const r=await fetch(`${API}/pedidos?usuarioId=${this.usuario.id}`)
        this.misPedidos=await r.json()
      } catch(e){}
    },
    getImg(m){ return m.imagen||IMGS[m.categoria]||IMGS['Sala'] },
    onImgError(e){ e.target.src=IMGS['Sala'] },
    setCategoria(cat){
      this.filtroCategoria=cat==='Todas'?'':cat
      this.$refs.catalogo?.scrollIntoView({behavior:'smooth'})
    },
    nextSlide(){ this.slideActual=(this.slideActual+1)%this.slides.length },
    prevSlide(){ this.slideActual=(this.slideActual-1+this.slides.length)%this.slides.length },
    verDetalle(m){ this.muebleDetalle=m; this.modalActivo='detalle' },
    agregarCarrito(m){
      const item=this.carrito.find(x=>x.id===m.id)
      if(item){ item.qty++ } else { this.carrito.push({...m,qty:1}) }
      this.toast(`${m.nombre} agregado al carrito`)
    },
    cambiarQty({ item, delta }){ item.qty+=delta; if(item.qty<=0) this.carrito=this.carrito.filter(x=>x.id!==item.id) },
    quitarCarrito(id){ this.carrito=this.carrito.filter(x=>x.id!==id) },
    iniciarCompra(){
      this.checkout.nombre=this.usuario.nombre
      this.pasoCheckout=1
      this.modalActivo='checkout'
    },
    irPaso2(){
      if(!this.checkout.direccion||!this.checkout.ciudad||!this.checkout.telefono){ this.toast('⚠️ Completa los datos de envío'); return }
      this.pasoCheckout=2
    },
    async confirmarCompra(){
      if(this.checkout.metodo==='tarjeta'&&(!this.checkout.tarjeta||!this.checkout.vencimiento||!this.checkout.cvv)){
        this.toast('⚠️ Completa los datos de tu tarjeta'); return
      }
      this.comprando=true
      const pedido={
        usuarioId:this.usuario.id,
        usuario:this.usuario.nombre,
        items:this.carrito.map(i=>({id:i.id,nombre:i.nombre,qty:i.qty,precio:i.precio})),
        total:this.totalPrecio,
        envio:{...this.checkout},
        estado:'✅ Confirmado',
        fecha:new Date().toLocaleDateString('es-MX',{day:'2-digit',month:'long',year:'numeric'})
      }
      try{
        const r=await fetch(`${API}/pedidos`,{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(pedido)})
        const saved=await r.json()
        this.ultimoPedido=saved.id
        this.ultimoTotal=this.totalPrecio
        this.ultimosItems=[...this.carrito]
        this.carrito=[]
        this.modalActivo='exito'
        await this.cargarPedidos()
      } catch(e){ this.toast('⚠️ Error al procesar el pedido') }
      this.comprando=false
    },
    verPedidos(){ this.modalActivo='pedidos' },
    abrirAdmin(m){
      if(!this.esAdmin) return
      if(m){ this.editandoId=m.id; this.form={...m,imagen:m.imagen||''} }
      else { this.editandoId=null; this.form={nombre:'',categoria:'Sala',precio:0,stock:0,descripcion:'',imagen:''} }
      this.modalActivo='admin'
    },
    cerrarAdmin(){ this.modalActivo=null; this.editandoId=null },
    async guardar(formData){
      if(!this.esAdmin) return
      if(!formData.nombre||!formData.precio||!formData.stock){ this.toast('⚠️ Completa los campos'); return }
      const headers={'Content-Type':'application/json'}; const body=JSON.stringify(formData)
      try{
        if(this.editandoId){ await fetch(`${API}/muebles/${this.editandoId}`,{method:'PUT',headers,body}); this.toast('Mueble actualizado ✓') }
        else { await fetch(`${API}/muebles`,{method:'POST',headers,body}); this.toast('Mueble agregado ✓') }
        this.cerrarAdmin(); await this.cargar()
      } catch(e){ this.toast('⚠️ Error al guardar') }
    },
    confirmarEliminar(m){ this.confirmItem=m; this.modalActivo='confirmar' },
    async eliminar(id){
      if(!this.esAdmin) return
      this.modalActivo=null
      try{ await fetch(`${API}/muebles/${id}`,{method:'DELETE'}); this.carrito=this.carrito.filter(x=>x.id!==id); this.toast('Mueble eliminado'); await this.cargar() }
      catch(e){ this.toast('⚠️ Error al eliminar') }
    },
    toast(msg){ this.toastMsg=msg; this.toastVisible=true; setTimeout(()=>{ this.toastVisible=false },2800) }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Fraunces:wght@600;700&display=swap');
*{box-sizing:border-box;margin:0;padding:0}
body{font-family:'Inter',sans-serif;background:#f4f1ec;color:#1c1c1c}
.login-screen{min-height:100vh;display:flex;align-items:center;justify-content:center;position:relative}
.login-bg{position:absolute;inset:0;background:url('https://images.unsplash.com/photo-1555041469-a586c61ea9bc?w=1400&q=80') center/cover;filter:brightness(0.3)}
.login-card{position:relative;background:white;border-radius:20px;padding:40px;width:100%;max-width:420px;box-shadow:0 24px 64px rgba(0,0,0,0.25)}
.login-brand{display:flex;align-items:center;gap:12px;margin-bottom:28px}
.brand-logo{width:40px;height:40px;background:#2d6a4f;color:white;border-radius:10px;display:flex;align-items:center;justify-content:center;font-family:'Fraunces',serif;font-size:16px;font-weight:700}
.login-title{font-family:'Fraunces',serif;font-size:18px;font-weight:700}
.login-sub{font-size:10px;color:#aaa;letter-spacing:1px}
.login-heading{font-family:'Fraunces',serif;font-size:28px;font-weight:700;margin-bottom:6px}
.login-desc{font-size:14px;color:#888;margin-bottom:24px}
.login-form{display:flex;flex-direction:column;gap:14px}
.lgroup{display:flex;flex-direction:column;gap:5px}
.lgroup label{font-size:11px;color:#888;font-weight:600;text-transform:uppercase;letter-spacing:0.5px}
.lgroup input{padding:11px 14px;border:1px solid #e0e0e0;border-radius:10px;font-size:15px;font-family:'Inter',sans-serif;background:#fafafa}
.lgroup input:focus{outline:2px solid #2d6a4f;outline-offset:-1px;background:white}
.pass-wrap{position:relative}
.pass-wrap input{width:100%}
.pass-toggle{position:absolute;right:10px;top:50%;transform:translateY(-50%);background:none;border:none;cursor:pointer;font-size:16px}
.login-error{background:#fef2f2;color:#dc2626;padding:10px 14px;border-radius:8px;font-size:13px;font-weight:500}
.login-btn{padding:12px;background:#2d6a4f;color:white;border:none;border-radius:10px;font-size:15px;font-weight:600;cursor:pointer;font-family:'Inter',sans-serif}
.login-btn:hover{background:#1f5038}
.login-btn:disabled{background:#86c5a8;cursor:not-allowed}
.login-hints{margin-top:20px;padding:14px;background:#f8f6f2;border-radius:10px;border:1px solid #e8e3d8}
.hint{display:flex;align-items:center;gap:10px;padding:7px 0;font-size:13px;color:#666;cursor:pointer}
.hint:hover{color:#2d6a4f}
.hint-rol{background:#1c1c1c;color:white;padding:2px 8px;border-radius:4px;font-size:10px;font-weight:700}
.hint-rol.cliente{background:#2d6a4f}
.navbar{position:sticky;top:0;z-index:100;background:white;border-bottom:1px solid #e8e3d8;display:flex;align-items:center;gap:20px;padding:0 28px;height:62px;box-shadow:0 1px 4px rgba(0,0,0,0.05)}
.nav-brand{display:flex;align-items:center;gap:10px;min-width:190px}
.brand-logo-sm{width:34px;height:34px;background:#2d6a4f;color:white;border-radius:8px;display:flex;align-items:center;justify-content:center;font-family:'Fraunces',serif;font-size:14px;font-weight:700}
.brand-name{font-family:'Fraunces',serif;font-size:15px;font-weight:700;line-height:1}
.brand-sub{font-size:9px;color:#aaa;letter-spacing:1px}
.nav-cats{display:flex;gap:4px;flex:1}
.nav-cat{padding:6px 14px;border-radius:20px;border:none;background:transparent;cursor:pointer;font-size:14px;color:#666;font-family:'Inter',sans-serif;transition:all 0.15s}
.nav-cat:hover{color:#1c1c1c}
.nav-cat.active{background:#2d6a4f;color:white;font-weight:500}
.nav-right{display:flex;align-items:center;gap:10px}
.nav-search{display:flex;align-items:center;gap:8px;background:#f4f1ec;border-radius:20px;padding:6px 14px}
.nav-search input{border:none;background:transparent;font-size:14px;width:140px;outline:none;font-family:'Inter',sans-serif}
.btn-cart{position:relative;background:transparent;border:none;font-size:20px;cursor:pointer;padding:4px 8px}
.cart-badge{position:absolute;top:-4px;right:-4px;background:#dc2626;color:white;border-radius:50%;width:16px;height:16px;font-size:10px;display:flex;align-items:center;justify-content:center;font-weight:600}
.btn-nuevo{background:#2d6a4f;color:white;border:none;border-radius:8px;padding:7px 14px;cursor:pointer;font-size:13px;font-weight:500;font-family:'Inter',sans-serif}
.nav-user{position:relative;display:flex;align-items:center;gap:8px;cursor:pointer;padding:6px 10px;border-radius:10px;border:1px solid #e8e3d8;user-select:none}
.nav-user:hover{background:#f4f1ec}
.user-avatar{width:30px;height:30px;border-radius:50%;background:#2d6a4f;color:white;display:flex;align-items:center;justify-content:center;font-size:13px;font-weight:700}
.user-name{font-size:13px;font-weight:500;line-height:1}
.user-rol{font-size:10px;text-transform:uppercase;letter-spacing:0.5px;font-weight:600}
.user-rol.admin{color:#2d6a4f}
.user-rol.cliente{color:#b45309}
.user-menu{position:absolute;top:calc(100% + 6px);right:0;background:white;border:1px solid #e8e3d8;border-radius:12px;box-shadow:0 8px 24px rgba(0,0,0,0.1);min-width:200px;overflow:hidden;z-index:300}
.user-menu-email{padding:10px 16px;font-size:12px;color:#aaa;border-bottom:1px solid #f0ece4}
.user-menu-item{padding:10px 16px;font-size:14px;cursor:pointer}
.user-menu-item:hover{background:#f4f1ec}
.user-menu-item.danger{color:#dc2626}
.user-menu-sep{height:1px;background:#f0ece4;margin:4px 0}
.hero{position:relative;height:500px;overflow:hidden}
.hero-slide{position:absolute;inset:0;background-size:cover;background-position:center}
.hero-content{position:absolute;bottom:80px;left:60px;max-width:500px}
.hero-badge{background:#d4a017;color:white;padding:4px 14px;border-radius:20px;font-size:12px;font-weight:700;letter-spacing:0.5px}
.hero-title{font-family:'Fraunces',serif;font-size:46px;font-weight:700;color:white;margin:12px 0 8px;line-height:1.1;text-shadow:0 2px 12px rgba(0,0,0,0.3)}
.hero-desc{color:rgba(255,255,255,0.85);font-size:16px;margin-bottom:24px}
.hero-btn{background:#2d6a4f;color:white;border:none;padding:13px 28px;border-radius:10px;font-size:15px;font-weight:600;cursor:pointer;font-family:'Inter',sans-serif}
.hero-btn:hover{background:#1f5038}
.arrow{position:absolute;top:50%;transform:translateY(-50%);background:rgba(255,255,255,0.15);border:none;color:white;font-size:28px;width:44px;height:44px;border-radius:50%;cursor:pointer;backdrop-filter:blur(4px)}
.arrow.left{left:16px}
.arrow.right{right:16px}
.dots{position:absolute;bottom:22px;left:50%;transform:translateX(-50%);display:flex;gap:8px}
.dot{width:8px;height:8px;border-radius:50%;background:rgba(255,255,255,0.4);cursor:pointer}
.dot.active{background:white}
.fade-enter-active,.fade-leave-active{transition:opacity 0.6s}
.fade-enter-from,.fade-leave-to{opacity:0}
.stats{background:#1c1c1c;display:flex;justify-content:center}
.stat{flex:1;max-width:240px;padding:22px;text-align:center;border-right:1px solid rgba(255,255,255,0.08)}
.stat:last-child{border-right:none}
.stat-val{font-family:'Fraunces',serif;font-size:24px;font-weight:700;color:#d4a017}
.stat-label{font-size:11px;color:rgba(255,255,255,0.5);letter-spacing:1px;margin-top:2px}
.catalogo{max-width:1200px;margin:0 auto;padding:36px 24px}
.cat-header{margin-bottom:20px}
.cat-count{font-size:13px;color:#888;font-weight:500}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(260px,1fr));gap:20px}
.card{background:white;border-radius:14px;overflow:hidden;box-shadow:0 1px 4px rgba(0,0,0,0.06);transition:transform 0.2s,box-shadow 0.2s}
.card:hover{transform:translateY(-3px);box-shadow:0 8px 24px rgba(0,0,0,0.1)}
.card-img-wrap{position:relative;overflow:hidden;height:200px}
.card-img{width:100%;height:100%;object-fit:cover;transition:transform 0.4s}
.card:hover .card-img{transform:scale(1.05)}
.badge-stock{position:absolute;top:10px;left:10px;background:#d4a017;color:white;padding:3px 10px;border-radius:20px;font-size:11px;font-weight:600}
.badge-agotado{position:absolute;top:10px;left:10px;background:#dc2626;color:white;padding:3px 10px;border-radius:20px;font-size:11px;font-weight:600}
.card-edit,.card-del{position:absolute;top:8px;right:8px;background:rgba(255,255,255,0.92);border:none;border-radius:6px;padding:4px 8px;cursor:pointer;font-size:13px;opacity:0;transition:opacity 0.2s}
.card-del{right:52px}
.card:hover .card-edit,.card:hover .card-del{opacity:1}
.card-body{padding:16px}
.card-cat{font-size:11px;font-weight:700;color:#2d6a4f;letter-spacing:0.5px;margin-bottom:4px}
.card-name{font-family:'Fraunces',serif;font-size:17px;font-weight:600;margin-bottom:6px;line-height:1.3}
.card-desc{font-size:13px;color:#888;line-height:1.5;margin-bottom:12px;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical;overflow:hidden}
.card-footer{display:flex;align-items:center;justify-content:space-between}
.card-price{font-size:20px;font-weight:700}
.card-price sup{font-size:13px;vertical-align:super}
.card-ver{background:#f4f1ec;border:none;padding:7px 16px;border-radius:8px;font-size:13px;font-weight:500;cursor:pointer}
.card-ver:hover{background:#e8e3d8}
.empty{text-align:center;padding:60px;color:#aaa;font-size:16px}
.overlay{position:fixed;inset:0;background:rgba(0,0,0,0.45);display:flex;align-items:center;justify-content:center;z-index:200;padding:16px}
.modal-detalle,.modal-carrito,.modal-admin,.modal-checkout,.modal-pedidos{background:white;border-radius:18px;width:100%;overflow:hidden}
.modal-detalle{max-width:780px}
.modal-carrito{max-width:560px;max-height:85vh;overflow-y:auto}
.modal-admin{max-width:560px}
.modal-checkout{max-width:620px;max-height:85vh;overflow-y:auto}
.modal-pedidos{max-width:560px;max-height:80vh;overflow-y:auto}
.modal-breadcrumb{padding:16px 24px 0;font-size:13px;color:#aaa}
.modal-header{display:flex;align-items:center;justify-content:space-between;padding:16px 24px}
.modal-header h2{font-family:'Fraunces',serif;font-size:22px;font-weight:700}
.modal-close{background:transparent;border:1px solid #e0e0e0;border-radius:8px;padding:6px 12px;cursor:pointer;font-size:16px}
.modal-body{display:grid;grid-template-columns:1fr 1fr;gap:24px;padding:0 24px 20px}
.modal-img{width:100%;height:280px;object-fit:cover;border-radius:12px}
.modal-brand{color:#d4a017;font-size:11px;font-weight:700;letter-spacing:1px;margin-bottom:12px}
.modal-desc{font-size:15px;color:#555;line-height:1.6;margin-bottom:20px}
.modal-stats{display:grid;grid-template-columns:1fr 1fr 1fr;gap:10px}
.mstat{background:#f4f1ec;border-radius:10px;padding:12px}
.mstat-label{font-size:10px;color:#aaa;font-weight:600;letter-spacing:0.5px;margin-bottom:4px}
.mstat-val{font-size:15px;font-weight:700}
.modal-actions{display:flex;justify-content:flex-end;gap:10px;padding:16px 24px;border-top:1px solid #f0ece4}
.btn-cerrar{padding:10px 20px;background:#f4f1ec;border:none;border-radius:8px;cursor:pointer;font-size:14px;font-weight:500;font-family:'Inter',sans-serif}
.btn-agregar{padding:10px 24px;background:#2d6a4f;color:white;border:none;border-radius:8px;cursor:pointer;font-size:14px;font-weight:500;font-family:'Inter',sans-serif}
.btn-agregar:hover{background:#1f5038}
.btn-agregar:disabled{background:#aaa;cursor:not-allowed}
.btn-danger{padding:10px 24px;background:#dc2626;color:white;border:none;border-radius:8px;cursor:pointer;font-size:14px;font-weight:500;font-family:'Inter',sans-serif}
.carrito-lista{padding:0 20px}
.carrito-item{display:flex;align-items:center;gap:12px;padding:12px 0;border-bottom:1px solid #f0ece4}
.ci-img{width:56px;height:56px;object-fit:cover;border-radius:8px}
.ci-info{flex:1}
.ci-name{font-size:14px;font-weight:500;margin-bottom:2px}
.ci-price{font-size:13px;color:#888}
.qty-ctrl{display:flex;align-items:center;gap:8px}
.qty-ctrl button{width:28px;height:28px;border-radius:6px;border:1px solid #e0e0e0;background:#f4f1ec;cursor:pointer;font-size:16px;display:flex;align-items:center;justify-content:center}
.qty-ctrl span{font-size:15px;font-weight:600;min-width:20px;text-align:center}
.ci-sub{font-size:15px;font-weight:700;min-width:70px;text-align:right}
.ci-del{background:transparent;border:none;cursor:pointer;color:#ccc;font-size:16px;padding:4px}
.ci-del:hover{color:#dc2626}
.carrito-footer{padding:16px 20px;border-top:1px solid #f0ece4}
.carrito-total{display:flex;justify-content:space-between;align-items:center;margin-bottom:14px;font-size:15px;color:#888;font-weight:500}
.total-num{font-size:24px;font-weight:700;color:#1c1c1c}
.btn-comprar{width:100%;padding:13px;background:#2d6a4f;color:white;border:none;border-radius:10px;font-size:15px;font-weight:600;cursor:pointer;font-family:'Inter',sans-serif}
.btn-comprar:hover{background:#1f5038}
.btn-comprar:disabled{background:#aaa;cursor:not-allowed}
.checkout-body{padding:0 24px 24px}
.checkout-section{margin-bottom:20px}
.checkout-section-title{font-family:'Fraunces',serif;font-size:17px;font-weight:700;margin-bottom:14px}
.checkout-grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.checkout-resumen{background:#f4f1ec;border-radius:12px;padding:16px;margin-bottom:16px}
.resumen-item{display:flex;justify-content:space-between;font-size:13px;color:#666;padding:4px 0}
.resumen-total{display:flex;justify-content:space-between;font-size:15px;font-weight:700;padding-top:10px;margin-top:8px;border-top:1px solid #e8e3d8}
.checkout-actions{display:flex;justify-content:flex-end;gap:10px;padding-top:16px;border-top:1px solid #f0ece4}
.metodos-pago{display:flex;flex-direction:column;gap:8px;margin-bottom:16px}
.metodo{padding:12px 16px;border:1px solid #e0e0e0;border-radius:10px;cursor:pointer;font-size:14px;display:flex;align-items:center;gap:10px;transition:all 0.15s}
.metodo:hover{border-color:#2d6a4f}
.metodo.active{border-color:#2d6a4f;background:#f0f9f5;font-weight:500}
.tarjeta-form{display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-top:12px}
.info-pago{background:#f4f1ec;border-radius:10px;padding:14px;font-size:13px;color:#666;line-height:1.8;margin-top:12px}
.modal-exito{background:white;border-radius:18px;padding:36px;width:100%;max-width:420px;text-align:center}
.exito-icon{width:64px;height:64px;background:#2d6a4f;color:white;border-radius:50%;display:flex;align-items:center;justify-content:center;font-size:28px;margin:0 auto 16px}
.exito-title{font-family:'Fraunces',serif;font-size:26px;font-weight:700;margin-bottom:8px}
.exito-desc{font-size:14px;color:#888;line-height:1.6;margin-bottom:20px}
.exito-detalle{background:#f4f1ec;border-radius:12px;padding:16px;text-align:left;margin-bottom:16px}
.exito-row{display:flex;justify-content:space-between;font-size:13px;color:#666;padding:4px 0}
.exito-total{display:flex;justify-content:space-between;font-size:15px;font-weight:700;padding-top:10px;margin-top:8px;border-top:1px solid #e8e3d8}
.pedidos-lista{padding:0 20px 20px;display:flex;flex-direction:column;gap:12px}
.pedido-card{background:#f4f1ec;border-radius:12px;padding:16px}
.pedido-header{display:flex;align-items:center;gap:10px;margin-bottom:8px}
.pedido-id{font-weight:600;font-size:14px}
.pedido-fecha{font-size:12px;color:#888;flex:1}
.pedido-status{font-size:12px;font-weight:600;color:#2d6a4f}
.pedido-items{font-size:13px;color:#666;margin-bottom:6px;display:flex;flex-wrap:wrap;gap:6px}
.pedido-items span{background:white;padding:3px 8px;border-radius:6px}
.pedido-total{font-size:14px;font-weight:700}
.admin-form{display:grid;grid-template-columns:1fr 1fr;gap:12px;padding:0 24px 16px}
.fgroup{display:flex;flex-direction:column;gap:4px}
.fgroup.full{grid-column:1/-1}
.fgroup label{font-size:11px;color:#aaa;font-weight:600;text-transform:uppercase;letter-spacing:0.5px}
.fgroup input,.fgroup select,.fgroup textarea{padding:9px 12px;border:1px solid #e0e0e0;border-radius:8px;font-size:14px;font-family:'Inter',sans-serif;background:#fafafa}
.fgroup input:focus,.fgroup select:focus,.fgroup textarea:focus{outline:2px solid #2d6a4f;outline-offset:-1px;background:white}
.fgroup textarea{resize:vertical;min-height:72px}
.modal-confirm{background:white;border-radius:16px;padding:28px;width:100%;max-width:360px}
.modal-confirm h3{font-family:'Fraunces',serif;font-size:18px;font-weight:700;margin-bottom:8px}
.modal-confirm p{font-size:14px;color:#888;margin-bottom:20px;line-height:1.5}
.toast{position:fixed;bottom:24px;right:24px;background:#1c1c1c;color:white;padding:12px 20px;border-radius:10px;font-size:14px;font-weight:500;opacity:0;transition:opacity 0.3s;pointer-events:none;z-index:999}
.toast.show{opacity:1}
</style>
