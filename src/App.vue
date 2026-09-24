<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from 'vue'
import {
  ArrowDown, ArrowLeft, ArrowRight, ArrowUpRight, Check, ChevronDown,
  Instagram, Menu, Phone, Play, Plus, X
} from 'lucide-vue-next'

const route = ref(location.hash.replace('#/', '') || 'home')
const menuOpen = ref(false)
const inquiryOpen = ref(false)
const submitted = ref(false)
const activeFaq = ref(0)
const currentProject = ref(0)
const activeStage = ref(0)
const scrollY = ref(0)
const scrollProgress = ref(0)
const activeGallery = ref(0)
const lightboxOpen = ref(false)

const vReveal = {
  mounted(el, binding) {
    el.classList.add('reveal')
    if (binding.value) el.style.setProperty('--reveal-delay', `${binding.value}ms`)
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        el.classList.add('is-visible')
        observer.disconnect()
      }
    }, { threshold: 0.12, rootMargin: '0px 0px -45px' })
    observer.observe(el)
  }
}

const services = [
  { number: '01', title: 'Luxury Pools & Spas', subtitle: 'The heart of your private resort.', image: '/images/service-pool.webp', copy: 'Custom geometry, architectural edges, integrated spas, tanning ledges, water features, smart lighting and timeless finishes—designed around the way you live.' },
  { number: '02', title: 'Outdoor Living Spaces', subtitle: 'Made for gathering.', image: '/images/service-outdoor.webp', copy: 'Outdoor kitchens, fire features, shaded lounges and dining spaces that feel like a natural extension of your home.' },
  { number: '03', title: 'Wellness Zones', subtitle: 'Your daily reset, steps from home.', image: '/images/service-wellness.webp', copy: 'Saunas, cold plunges, yoga decks, spas and quiet relaxation areas integrated into the landscape with privacy and intention.' },
  { number: '04', title: 'Play & Performance', subtitle: 'Space to move, play and connect.', image: '/images/service-play.webp', copy: 'Putting greens, basketball courts, play areas and entertainment zones designed beautifully for every generation.' },
]

const projects = [
  {
    slug: 'la-jolla-modern-haven', name: 'La Jolla Modern Haven', place: 'La Jolla, California', type: 'Pool · Fire · Wellness',
    image: '/images/project-la-jolla.png', detail: 'Infinity-edge pool, fire lounge and wellness deck.',
    story: 'A private hillside resort shaped around open views, architectural water and unhurried evenings. Every edge was designed to make the home and horizon feel like one continuous experience.',
    scope: ['Infinity-edge pool', 'Integrated spa', 'Fire lounge', 'Wellness deck'],
    gallery: ['/images/project-la-jolla.png', '/images/service-pool.webp', '/images/project-rancho.png', '/images/service-wellness.webp']
  },
  {
    slug: 'del-mar-coastal-retreat', name: 'Del Mar Coastal Retreat', place: 'Del Mar, California', type: 'Landscape · Sauna',
    image: '/images/project-del-mar.webp', detail: 'A coastal landscape with a private sauna retreat.',
    story: 'A calm coastal garden where wellness is woven into the landscape. Soft planting, warm timber and secluded recovery spaces turn an overlooked yard into a daily ritual.',
    scope: ['Private sauna', 'Cold plunge', 'Coastal planting', 'Relaxation deck'],
    gallery: ['/images/project-del-mar.webp', '/images/service-wellness.webp', '/images/service-outdoor.webp', '/images/project-coronado.webp']
  },
  {
    slug: 'rancho-santa-fe-legacy', name: 'Rancho Santa Fe Legacy', place: 'Rancho Santa Fe, California', type: 'Pool · Outdoor Living',
    image: '/images/project-rancho.png', detail: 'A Mediterranean-inspired outdoor resort.',
    story: 'Designed for an estate that brings generations together, this expansive retreat balances classic Mediterranean character with the ease of modern outdoor living.',
    scope: ['Custom pool & spa', 'Dining pavilion', 'Fire features', 'Estate landscape'],
    gallery: Array.from({ length: 6 }, (_, i) => `/images/rancho-${i + 1}.webp`)
  },
  {
    slug: 'san-marcos-dream-yard', name: 'San Marcos Dream Yard', place: 'San Marcos, California', type: 'Pool · Kitchen · Fire',
    image: '/images/project-san-marcos.webp', detail: 'Stunning pool and spa, outdoor kitchen and fire pit.',
    story: 'A complete family retreat where swimming, cooking and gathering happen in one fluid environment. The design gives every moment its own place without losing connection.',
    scope: ['Pool & raised spa', 'Outdoor kitchen', 'Fire pit lounge', 'Landscape'],
    gallery: Array.from({ length: 6 }, (_, i) => `/images/san-marcos-${i + 1}.webp`)
  },
  {
    slug: 'coronado-oceanfront', name: 'Coronado Oceanfront', place: 'Coronado, California', type: 'Coastal Outdoor Living',
    image: '/images/project-coronado.webp', detail: 'Elevated coastal outdoor design.',
    story: 'An oceanfront setting refined for salt air, sunset dinners and effortless entertaining. Materials were selected to age beautifully while preserving the openness of the view.',
    scope: ['Oceanfront terrace', 'Outdoor kitchen', 'Custom hardscape', 'Coastal planting'],
    gallery: Array.from({ length: 6 }, (_, i) => `/images/coronado-${i + 1}.webp`)
  },
  {
    slug: 'scripps-ranch-resort', name: 'Scripps Ranch Resort', place: 'Scripps Ranch, California', type: 'Family · Pool',
    image: '/images/project-scripps.webp', detail: 'A resort-style family backyard with a custom waterslide.',
    story: 'Playful enough for the kids and considered enough for the adults, this backyard turns family time into a permanent vacation without compromising its architectural character.',
    scope: ['Resort pool', 'Custom waterslide', 'Spa', 'Family lounge'],
    gallery: Array.from({ length: 6 }, (_, i) => `/images/scripps-${i + 1}.webp`)
  },
  {
    slug: 'pacific-beach-oasis', name: 'Pacific Beach Oasis', place: 'Pacific Beach, California', type: 'Pool · Coastal Landscape',
    image: '/images/pacific-1.webp', detail: 'A custom pool oasis designed for relaxed coastal living.',
    story: 'A compact coastal property reimagined with confident geometry, layered greenery and a pool that makes every square foot feel generous.',
    scope: ['Custom pool', 'Integrated spa', 'Privacy planting', 'Pool deck'],
    gallery: Array.from({ length: 6 }, (_, i) => `/images/pacific-${i + 1}.webp`)
  },
  {
    slug: 'la-mesa-luxury-retreat', name: 'La Mesa Luxury Retreat', place: 'La Mesa, California', type: 'Full Backyard Transformation',
    image: '/images/la-mesa-1.webp', detail: 'A complete backyard transformation made for year-round life.',
    story: 'This full-property remodel creates a seamless sequence from home to pool, shaded lounge and garden—designed for the inland climate and year-round gatherings.',
    scope: ['Pool & spa', 'Covered lounge', 'Hardscape', 'Lighting design'],
    gallery: Array.from({ length: 6 }, (_, i) => `/images/la-mesa-${i + 1}.webp`)
  },
]

const journal = [
  { title: 'Pool placement 101: what most homeowners get wrong', category: 'Pool Design', image: '/images/blog-pool.png', date: 'June 2026' },
  { title: 'How to make a small San Diego backyard feel like a resort', category: 'Design Guide', image: '/images/blog-small.png', date: 'June 2026' },
  { title: 'Top backyard trends San Diego homeowners are choosing in 2026', category: 'Outdoor Living', image: '/images/blog-trends.jpeg', date: 'June 2026' },
]

const transformationStages = [
  {
    number: '01',
    label: 'Imagine',
    title: 'We uncover the life your yard could hold.',
    copy: 'We walk the property with you—studying its light, views, movement and untapped potential.',
    image: '/images/service-play.webp',
    note: 'Site + lifestyle discovery'
  },
  {
    number: '02',
    label: 'Visualize',
    title: 'See it before a single shovel moves.',
    copy: 'Your ideas become an immersive 3D environment where every line, material and sightline can be experienced and refined.',
    image: '/images/project-la-jolla.png',
    note: 'Immersive 3D design'
  },
  {
    number: '03',
    label: 'Transform',
    title: 'One team brings every detail to life.',
    copy: 'Permits, engineering, excavation and craftsmanship move in rhythm, with one accountable team guiding the build.',
    image: '/images/project-san-marcos.webp',
    note: 'White-glove construction'
  },
  {
    number: '04',
    label: 'Live',
    title: 'Your everyday becomes the destination.',
    copy: 'The final reveal is only the beginning: slow mornings, long dinners and years of life lived outside.',
    image: '/images/project-rancho.png',
    note: 'A resort that is entirely yours'
  }
]

const faqs = [
  ['What does a full backyard remodel cost in San Diego?', 'Our design-build projects generally begin around $80,000. Complete transformations vary based on site conditions, engineering, pool design, materials and outdoor features. Your consultation clarifies the real scope before you commit.'],
  ['Do you handle design, permits and construction?', 'Yes. One team manages 3D design, engineering, permits, construction, finishes, landscape and the final reveal under one roof.'],
  ['Where in San Diego do you work?', 'We serve San Diego and surrounding communities, including La Jolla, Del Mar, Rancho Santa Fe, Encinitas, Carlsbad, Poway, Coronado and Scripps Ranch.'],
  ['Can I see the design before construction?', 'Yes. Every project is presented as a detailed 3D rendering, so you can understand the layout, materials and feeling of the finished space before building begins.'],
]

const pageTitle = computed(() => ({
  projects: ['Selected work', 'Spaces with a story.'],
  about: ['Our studio', 'Built around how you live.'],
  journal: ['The journal', 'Ideas for life outside.'],
  contact: ['Start a conversation', 'Your backyard begins here.'],
}[route.value]))
const selectedProject = computed(() => {
  if (!route.value.startsWith('project/')) return null
  return projects.find(project => project.slug === route.value.split('/')[1]) || null
})

function go(page) {
  location.hash = `/${page}`
  menuOpen.value = false
}

function onHash() {
  route.value = location.hash.replace('#/', '') || 'home'
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function nextProject(direction = 1) {
  currentProject.value = (currentProject.value + direction + projects.length) % projects.length
}

function openProject(project) {
  activeGallery.value = 0
  go(`project/${project.slug}`)
}

function nextGallery(direction = 1) {
  if (!selectedProject.value) return
  const total = selectedProject.value.gallery.length
  activeGallery.value = (activeGallery.value + direction + total) % total
}

function onKeydown(event) {
  if (!lightboxOpen.value) return
  if (event.key === 'Escape') lightboxOpen.value = false
  if (event.key === 'ArrowLeft') nextGallery(-1)
  if (event.key === 'ArrowRight') nextGallery(1)
}

function submitForm() {
  submitted.value = true
  setTimeout(() => {
    submitted.value = false
    inquiryOpen.value = false
  }, 2600)
}

function onScroll() {
  scrollY.value = window.scrollY
  const height = document.documentElement.scrollHeight - window.innerHeight
  scrollProgress.value = height > 0 ? Math.min((window.scrollY / height) * 100, 100) : 0
}

watch(inquiryOpen, value => document.body.classList.toggle('no-scroll', value))
watch(lightboxOpen, value => document.body.classList.toggle('no-scroll', value))
watch(route, () => { activeGallery.value = 0 })
onMounted(() => {
  window.addEventListener('hashchange', onHash)
  window.addEventListener('scroll', onScroll, { passive: true })
  window.addEventListener('keydown', onKeydown)
  onScroll()
})
onUnmounted(() => {
  window.removeEventListener('hashchange', onHash)
  window.removeEventListener('scroll', onScroll)
  window.removeEventListener('keydown', onKeydown)
})
</script>

<template>
  <div class="site-shell">
    <div class="scroll-progress" :style="{ width: `${scrollProgress}%` }"></div>
    <header class="nav" :class="{ 'nav--light': route !== 'home' }">
      <button class="brand" aria-label="Home" @click="go('home')">
        <span class="brand-mark">E</span>
        <span class="brand-type">Everlasting<small>POOLS &amp; LANDSCAPE</small></span>
      </button>
      <nav class="desktop-nav" aria-label="Main navigation">
        <button @click="go('home')">Home</button>
        <button @click="go('projects')">Projects</button>
        <button @click="go('about')">About</button>
        <button @click="go('journal')">Journal</button>
      </nav>
      <div class="nav-actions">
        <a class="phone" href="tel:+18582503233"><Phone :size="15" /> (858) 250-3233</a>
        <button class="nav-cta" @click="inquiryOpen = true">Start a project <ArrowUpRight :size="15" /></button>
        <button class="menu-toggle" aria-label="Open menu" @click="menuOpen = true"><Menu /></button>
      </div>
    </header>

    <Transition name="fade">
      <div v-if="menuOpen" class="mobile-menu">
        <button class="menu-close" aria-label="Close menu" @click="menuOpen = false"><X /></button>
        <div class="mobile-brand">EVERLASTING</div>
        <nav>
          <button v-for="item in ['home', 'projects', 'about', 'journal', 'contact']" :key="item" @click="go(item)">
            {{ item }}
          </button>
        </nav>
        <a href="tel:+18582503233">(858) 250-3233</a>
      </div>
    </Transition>

    <main v-if="route === 'home'">
      <section class="hero" :style="{ '--hero-y': `${scrollY * 0.14}px` }">
        <img src="/images/hero.jpeg" alt="Luxury pool and outdoor living space in San Diego" />
        <div class="hero-shade"></div>
        <div class="hero-grid" aria-hidden="true"><i></i><i></i><i></i><i></i></div>
        <div class="hero-content">
          <p class="eyebrow light">San Diego · Design + Build</p>
          <h1>Your private resort,<br /><em>designed for life.</em></h1>
          <p class="hero-copy">Luxury pools, landscapes, and outdoor wellness experiences—crafted for those who value beauty, family, and legacy.</p>
          <button class="button button--ivory" @click="inquiryOpen = true">Begin your experience <ArrowUpRight :size="17" /></button>
        </div>
        <div class="hero-proof">
          <div class="stars">★★★★★</div>
          <span><strong>300+</strong> five-star reviews</span>
        </div>
        <a class="hero-scroll" href="#approach"><span>Discover</span><ArrowDown :size="17" /></a>
      </section>

      <div class="moving-line" aria-hidden="true">
        <div>DESIGN FOR LIFE <i></i> POOLS + SPAS <i></i> LANDSCAPE <i></i> OUTDOOR WELLNESS <i></i> SAN DIEGO <i></i> DESIGN FOR LIFE <i></i> POOLS + SPAS <i></i> LANDSCAPE <i></i></div>
      </div>

      <section id="approach" class="intro section" v-reveal>
        <div class="section-kicker"><span>01</span> Our approach</div>
        <div class="intro-grid">
          <h2>We don’t just build backyards.<br /><em>We shape how life feels.</em></h2>
          <div>
            <p class="lead">Every project begins with your story—how you gather, restore, celebrate, and connect.</p>
            <p>As a family-owned San Diego design-build studio, we unite architecture, landscape and craftsmanship into one seamless experience. The result is not a collection of features. It is a place that belongs to you.</p>
            <button class="text-link" @click="go('about')">Meet the people behind the work <ArrowRight :size="16" /></button>
          </div>
        </div>
      </section>

      <section class="transformation">
        <div class="transformation-visual">
          <Transition name="stage" mode="out-in">
            <img :key="activeStage" :src="transformationStages[activeStage].image" :alt="transformationStages[activeStage].title" />
          </Transition>
          <div class="design-scan" aria-hidden="true"></div>
          <div class="plan-overlay" aria-hidden="true">
            <span></span><span></span><span></span>
            <b>EVERLASTING / DESIGN STUDY</b>
          </div>
          <p>{{ transformationStages[activeStage].note }}</p>
        </div>
        <div class="transformation-content">
          <div class="section-kicker light"><span>02</span> The transformation</div>
          <Transition name="stage-copy" mode="out-in">
            <div :key="activeStage" class="stage-copy">
              <span>{{ transformationStages[activeStage].number }} / 04 · {{ transformationStages[activeStage].label }}</span>
              <h2>{{ transformationStages[activeStage].title }}</h2>
              <p>{{ transformationStages[activeStage].copy }}</p>
            </div>
          </Transition>
          <div class="stage-nav">
            <button
              v-for="(stage, i) in transformationStages"
              :key="stage.label"
              :class="{ active: activeStage === i }"
              @click="activeStage = i"
            >
              <span></span>{{ stage.label }}
            </button>
          </div>
        </div>
      </section>

      <section class="process section" v-reveal>
        <div class="process-heading">
          <div class="section-kicker light"><span>03</span> The experience</div>
          <h2>From first thought<br />to <em>favorite place.</em></h2>
        </div>
        <div class="process-steps">
          <article v-for="(step, i) in [
            ['Discovery', 'We walk your space, listen closely, and learn how you want to live outdoors.'],
            ['Design experience', 'Your vision takes form through immersive 3D renderings and considered material selections.'],
            ['Planning', 'Engineering, permits, selections and schedules—thoughtfully handled by one team.'],
            ['Build & reveal', 'Precision craftsmanship, clear communication, and a final space made to last.']
          ]" :key="step[0]" v-reveal="i * 90">
            <span>0{{ i + 1 }}</span>
            <div><h3>{{ step[0] }}</h3><p>{{ step[1] }}</p></div>
          </article>
        </div>
        <button class="button button--outline" @click="inquiryOpen = true">Start my design experience <ArrowUpRight :size="17" /></button>
      </section>

      <section class="services section">
        <div class="section-kicker"><span>04</span> What we create</div>
        <div class="services-title">
          <h2>One vision.<br /><em>Every detail.</em></h2>
          <p>Integrated spaces designed as a complete environment, never as an afterthought.</p>
        </div>
        <div class="service-list">
          <article v-for="(service, i) in services" :key="service.title" class="service-row" v-reveal="i * 70">
            <span class="service-number">{{ service.number }}</span>
            <div class="service-image"><img :src="service.image" :alt="service.title" /></div>
            <div class="service-copy">
              <h3>{{ service.title }}</h3>
              <p class="service-subtitle">{{ service.subtitle }}</p>
              <p class="service-detail">{{ service.copy }}</p>
            </div>
            <ArrowUpRight class="service-arrow" />
          </article>
        </div>
      </section>

      <section class="featured">
        <div class="featured-copy">
          <div class="section-kicker light"><span>05</span> Selected work</div>
          <Transition name="project" mode="out-in">
            <div :key="currentProject">
              <p class="project-type">{{ projects[currentProject].type }}</p>
              <h2>{{ projects[currentProject].name }}</h2>
              <p>{{ projects[currentProject].detail }}</p>
              <span class="project-place">{{ projects[currentProject].place }}</span>
            </div>
          </Transition>
          <div class="project-controls">
            <button aria-label="Previous project" @click="nextProject(-1)"><ArrowLeft /></button>
            <span>0{{ currentProject + 1 }} / 0{{ projects.length }}</span>
            <button aria-label="Next project" @click="nextProject(1)"><ArrowRight /></button>
          </div>
        </div>
        <div class="featured-image">
          <Transition name="project" mode="out-in">
            <img :key="currentProject" :src="projects[currentProject].image" :alt="projects[currentProject].name" />
          </Transition>
          <button class="image-action" @click="go('projects')">View all projects <ArrowUpRight :size="17" /></button>
        </div>
      </section>

      <section class="testimonial section" v-reveal>
        <div class="quote-mark">“</div>
        <blockquote>They didn’t just build us a pool.<br /><em>They built our lifestyle.</em></blockquote>
        <p>— K.L., La Jolla</p>
        <div class="review-line"><span>★★★★★</span> 5.0 across 300+ reviews</div>
      </section>

      <section class="journal-preview section">
        <div class="journal-heading">
          <div><div class="section-kicker"><span>06</span> Field notes</div><h2>Ideas for <em>life outside.</em></h2></div>
          <button class="text-link" @click="go('journal')">Explore the journal <ArrowRight :size="16" /></button>
        </div>
        <div class="article-grid">
          <article v-for="(article, i) in journal" :key="article.title" v-reveal="i * 100">
            <div class="article-image"><img :src="article.image" :alt="article.title" /><ArrowUpRight /></div>
            <p>{{ article.category }} · {{ article.date }}</p>
            <h3>{{ article.title }}</h3>
          </article>
        </div>
      </section>
    </main>

    <main v-else class="inner-page">
      <section v-if="!selectedProject" class="page-hero">
        <div class="section-kicker"><span>Everlasting</span> San Diego</div>
        <h1>{{ pageTitle?.[0] }}<br /><em>{{ pageTitle?.[1] }}</em></h1>
      </section>

      <section v-if="route === 'projects'" class="projects-page section">
        <div class="filter-row"><span>All projects</span><span>Pool + spa</span><span>Outdoor living</span><span>Landscape</span></div>
        <div class="projects-grid">
          <article
            v-for="(project, i) in projects"
            :key="project.name"
            :class="{ large: i === 0 || i === 3 }"
            role="button"
            tabindex="0"
            @click="openProject(project)"
            @keydown.enter="openProject(project)"
          >
            <div><img :src="project.image" :alt="project.name" /><span>0{{ i + 1 }}</span></div>
            <p>{{ project.type }}</p><h2>{{ project.name }} <ArrowUpRight :size="20" /></h2><small>{{ project.place }}</small>
          </article>
        </div>
      </section>

      <section v-if="selectedProject" class="project-detail">
        <div class="project-detail-head">
          <button class="back-link" @click="go('projects')"><ArrowLeft :size="16" /> All projects</button>
          <div class="project-index">Case study · 0{{ projects.indexOf(selectedProject) + 1 }}</div>
          <h1>{{ selectedProject.name }}</h1>
          <div class="project-meta">
            <span>{{ selectedProject.place }}</span>
            <span>{{ selectedProject.type }}</span>
          </div>
        </div>

        <div class="project-carousel">
          <Transition name="gallery" mode="out-in">
            <button :key="activeGallery" class="carousel-image" aria-label="Open fullscreen gallery" @click="lightboxOpen = true">
              <img :src="selectedProject.gallery[activeGallery]" :alt="`${selectedProject.name}, image ${activeGallery + 1}`" />
              <span><Plus :size="16" /> View fullscreen</span>
            </button>
          </Transition>
          <div class="carousel-ui">
            <div class="carousel-count">
              <strong>0{{ activeGallery + 1 }}</strong>
              <span>/ 0{{ selectedProject.gallery.length }}</span>
            </div>
            <div class="carousel-progress"><i :style="{ width: `${((activeGallery + 1) / selectedProject.gallery.length) * 100}%` }"></i></div>
            <div class="carousel-arrows">
              <button aria-label="Previous image" @click="nextGallery(-1)"><ArrowLeft /></button>
              <button aria-label="Next image" @click="nextGallery(1)"><ArrowRight /></button>
            </div>
          </div>
        </div>

        <div class="project-story section" v-reveal>
          <div>
            <div class="section-kicker"><span>01</span> The story</div>
            <h2>Designed around<br /><em>the life within it.</em></h2>
          </div>
          <div>
            <p class="lead">{{ selectedProject.detail }}</p>
            <p>{{ selectedProject.story }}</p>
          </div>
          <div class="project-scope">
            <span>Project scope</span>
            <p v-for="item in selectedProject.scope" :key="item"><Check :size="13" /> {{ item }}</p>
          </div>
        </div>

        <div class="gallery-rail-wrap">
          <div class="rail-heading">
            <div><span>Explore the space</span><h2>Every angle, <em>considered.</em></h2></div>
            <p>Drag or tap to move through the project.</p>
          </div>
          <div class="gallery-rail">
            <button
              v-for="(image, i) in selectedProject.gallery"
              :key="image"
              :class="{ active: activeGallery === i }"
              @click="activeGallery = i"
            >
              <img :src="image" :alt="`${selectedProject.name} view ${i + 1}`" />
              <span>0{{ i + 1 }}</span>
            </button>
          </div>
        </div>

        <div class="next-project">
          <p>Continue exploring</p>
          <button @click="openProject(projects[(projects.indexOf(selectedProject) + 1) % projects.length])">
            <span>Next project</span>
            <strong>{{ projects[(projects.indexOf(selectedProject) + 1) % projects.length].name }}</strong>
            <ArrowRight />
          </button>
        </div>
      </section>

      <section v-if="route === 'about'" class="about-page section">
        <div class="about-opening">
          <p class="eyebrow">Family owned · Purpose led</p>
          <h2>Meet Dani &amp; Joe.</h2>
          <div class="about-copy">
            <p class="lead">We’re a family-owned and operated design-build company with one mission: to create outdoor spaces that truly improve the way you live.</p>
            <p>Ever since we started, our passion has been transforming ordinary backyards into places where families connect, relax, and make memories. Every design we create is personal, intentional, and crafted with the same care we’d give our own home.</p>
            <p>At Everlasting, you’re not just another project. You’re part of our extended family. We listen, we dream with you, and we walk with you through every step—from concept to completion.</p>
          </div>
        </div>
        <div class="values-banner">
          <p>Our belief</p>
          <blockquote>Your yard should make you feel something. Calm, inspired, proud, <em>at peace.</em></blockquote>
        </div>
        <div class="stats">
          <div><strong>300+</strong><span>Five-star reviews</span></div>
          <div><strong>5.0</strong><span>Average rating</span></div>
          <div><strong>01</strong><span>Accountable team</span></div>
          <div><strong>#1118419</strong><span>CA licensed</span></div>
        </div>
      </section>

      <section v-if="route === 'journal'" class="journal-page section">
        <div class="article-grid">
          <article v-for="(article, i) in [...journal, ...journal]" :key="i">
            <div class="article-image"><img :src="article.image" :alt="article.title" /><ArrowUpRight /></div>
            <p>{{ article.category }} · {{ article.date }}</p>
            <h3>{{ i > 2 ? ['The seven elements of a luxury backyard in San Diego', 'Why your pool needs a water feature', 'How to design a backyard around the way you live'][i - 3] : article.title }}</h3>
          </article>
        </div>
      </section>

      <section v-if="route === 'contact'" class="contact-page section">
        <div class="contact-intro"><p class="lead">Tell us what you imagine. We’ll help you see what is possible.</p><p>Begin with a complimentary design consultation. Our team will learn about your property, priorities and the way you want to live outdoors.</p></div>
        <form class="contact-form" @submit.prevent="submitForm">
          <label>Full name<input required placeholder="Your name" /></label>
          <label>Email address<input required type="email" placeholder="you@email.com" /></label>
          <label>Phone number<input required type="tel" placeholder="(858) 000-0000" /></label>
          <label>Project location<input placeholder="City or neighborhood" /></label>
          <label class="full">Tell us about your project<textarea rows="4" placeholder="What are you dreaming of?"></textarea></label>
          <button class="button button--dark" type="submit">{{ submitted ? 'Thank you — we’ll be in touch' : 'Send my inquiry' }} <Check v-if="submitted" :size="17" /><ArrowUpRight v-else :size="17" /></button>
        </form>
        <div class="contact-details">
          <div><span>Call</span><a href="tel:+18582503233">+1 (858) 250-3233</a></div>
          <div><span>Email</span><a href="mailto:info@everlastingpoolsandlandscape.com">info@everlastingpoolsandlandscape.com</a></div>
          <div><span>Visit</span><p>910 Grand Ave #207<br />San Diego, CA 92109</p></div>
        </div>
      </section>
    </main>

    <section v-if="route !== 'contact'" class="faq section">
      <div><div class="section-kicker"><span>07</span> Good to know</div><h2>Your questions,<br /><em>answered.</em></h2></div>
      <div class="faq-list">
        <article v-for="(faq, i) in faqs" :key="faq[0]" :class="{ open: activeFaq === i }">
          <button @click="activeFaq = activeFaq === i ? -1 : i"><span>{{ faq[0] }}</span><Plus :size="20" /></button>
          <div class="faq-answer"><p>{{ faq[1] }}</p></div>
        </article>
      </div>
    </section>

    <section class="closing">
      <img src="/images/project-rancho.png" alt="Luxury backyard in Rancho Santa Fe" />
      <div class="closing-shade"></div>
      <div>
        <p class="eyebrow light">Your home. Reimagined outdoors.</p>
        <h2>Let’s create the place<br />you’ll <em>never want to leave.</em></h2>
        <button class="button button--ivory" @click="inquiryOpen = true">Book a free consultation <ArrowUpRight :size="17" /></button>
      </div>
    </section>

    <footer>
      <div class="footer-top">
        <div class="footer-brand"><span class="brand-mark">E</span><h2>Everlasting<small>POOLS &amp; LANDSCAPE</small></h2></div>
        <p>Building luxury outdoor living experiences across San Diego.</p>
      </div>
      <div class="footer-grid">
        <div><span>Explore</span><button @click="go('home')">Home</button><button @click="go('projects')">Projects</button><button @click="go('about')">About</button><button @click="go('journal')">Journal</button></div>
        <div><span>Services</span><p>Custom pools &amp; spas</p><p>Outdoor living</p><p>Landscape design</p><p>Wellness backyards</p></div>
        <div><span>Visit</span><p>910 Grand Ave #207<br />San Diego, CA 92109</p><a href="tel:+18582503233">(858) 250-3233</a></div>
        <div><span>Follow</span><a href="https://www.instagram.com/" target="_blank"><Instagram :size="17" /> Instagram</a><p>CA License #1118419</p></div>
      </div>
      <div class="footer-bottom"><span>© 2026 Everlasting Pools &amp; Landscape</span><span>Privacy policy</span><span>Designed for life in San Diego</span></div>
    </footer>

    <Transition name="modal">
      <div v-if="inquiryOpen" class="modal">
        <button class="modal-backdrop" aria-label="Close" @click="inquiryOpen = false"></button>
        <div class="modal-card">
          <button class="modal-close" @click="inquiryOpen = false"><X /></button>
          <p class="eyebrow">Complimentary design consultation</p>
          <h2>Tell us about<br /><em>your vision.</em></h2>
          <form v-if="!submitted" @submit.prevent="submitForm">
            <label>Your name<input required /></label>
            <label>Email address<input required type="email" /></label>
            <label>Phone number<input required type="tel" /></label>
            <label>What would you like to create?<select required><option value="">Select your project</option><option>Complete backyard transformation</option><option>Custom pool &amp; spa</option><option>Outdoor living &amp; kitchen</option><option>Wellness backyard</option><option>Pool renovation</option></select></label>
            <button class="button button--dark">Request my consultation <ArrowUpRight :size="17" /></button>
          </form>
          <div v-else class="success"><Check /><h3>Thank you.</h3><p>Your vision is in good hands. We’ll be in touch soon.</p></div>
        </div>
      </div>
    </Transition>

    <Transition name="lightbox">
      <div v-if="lightboxOpen && selectedProject" class="lightbox">
        <div class="lightbox-top">
          <span>{{ selectedProject.name }}</span>
          <span>0{{ activeGallery + 1 }} / 0{{ selectedProject.gallery.length }}</span>
          <button aria-label="Close gallery" @click="lightboxOpen = false"><X /></button>
        </div>
        <Transition name="gallery" mode="out-in">
          <img :key="activeGallery" :src="selectedProject.gallery[activeGallery]" :alt="selectedProject.name" />
        </Transition>
        <button class="lightbox-prev" aria-label="Previous image" @click="nextGallery(-1)"><ArrowLeft /></button>
        <button class="lightbox-next" aria-label="Next image" @click="nextGallery(1)"><ArrowRight /></button>
        <div class="lightbox-strip">
          <button v-for="(image, i) in selectedProject.gallery" :key="image" :class="{ active: activeGallery === i }" @click="activeGallery = i">
            <img :src="image" alt="" />
          </button>
        </div>
      </div>
    </Transition>
  </div>
</template>
