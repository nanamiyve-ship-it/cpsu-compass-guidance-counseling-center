<script setup>
import { computed, ref } from 'vue'

const page = ref('Home')
const darkMode = ref(localStorage.getItem('compass-theme') === 'dark')
const mobileMenuOpen = ref(false)
const counselingMenuOpen = ref(false)
const showBooking = ref(false)
const showEmergency = ref(false)
const showSurvey = ref(false)
const showNotifications = ref(false)
const showPassword = ref(false)
const loginError = ref('')
const loginSuccess = ref(false)
const selectedRole = ref('Student')
const activeFilter = ref('All')
const announcementSearch = ref('')
const faqSearch = ref('')
const bookingMessage = ref('')
const concernMessage = ref('')
const moodMessage = ref('')
const surveyMessage = ref('')
const surveyRating = ref('')
const moodHistory = ref(JSON.parse(localStorage.getItem('compass-moods') || '[]'))
const appointmentHistory = ref(JSON.parse(localStorage.getItem('compass-appointments') || '[]'))
const booking = ref({ counselor: 'Any available counselor', date: '', time: '' })
const counselorRequests = ref([
  { student: 'Demo student A', detail: 'Counseling request · Nov 10, 2026', status: 'Pending' },
  { student: 'Demo student B', detail: 'Online counseling · Nov 11, 2026', status: 'Pending' },
])
const concern = ref('')
const mood = ref('')
const login = ref({ id: '', password: '', remember: false })

const navigation = ['Home', 'About Us', 'Counseling', 'Forms', 'Resources', 'Announcements', 'FAQ', 'Login']
const announcements = [
  { category: 'Announcement', date: 'Nov 02, 2026', title: 'Wellness Week is coming to campus', text: 'Join us for a week of mindful moments, creative workshops, and friendly conversations about student wellbeing.', tag: 'Campus life' },
  { category: 'Scholarship', date: 'Oct 28, 2026', title: 'Student assistance applications are open', text: 'Find out about available financial assistance and the documents to prepare before applying.', tag: 'Opportunities' },
  { category: 'Announcement', date: 'Oct 19, 2026', title: 'Updated counseling office hours', text: 'The Guidance Center welcomes walk-ins Monday to Friday. Schedule an appointment for a longer session.', tag: 'Office update' },
]
const faqs = [
  { question: 'Is counseling confidential?', answer: 'Yes. Counseling conversations are handled with care and privacy, subject to applicable safety, legal, and university policies. Please ask your counselor if you have questions about confidentiality.' },
  { question: 'How do I book an appointment?', answer: 'Select “Schedule an appointment,” choose your preferred counselor, date, and available time, then submit your request. In this demo, your request is stored only in this browser.' },
  { question: 'Can I visit without an appointment?', answer: 'Yes. Walk-ins are welcome during office hours, subject to counselor availability. If possible, call the office before visiting.' },
  { question: 'What should I expect in my first session?', answer: 'Your counselor will explain the process, listen to what brings you in, and work with you on next steps. You do not need to prepare anything in advance.' },
  { question: 'Where is the Guidance Center?', answer: 'The center is on the CPSU San Carlos Campus. Please confirm the current office location with campus staff before visiting.' },
]
const resources = [
  { icon: '✳', type: 'Wellbeing', title: 'A gentler way to manage stress', text: 'Small grounding practices to help you reset when everything feels like a lot.', time: '4 min read', tint: 'mint' },
  { icon: '◷', type: 'Study skills', title: 'Build a study routine that sticks', text: 'Make your next study session easier to start with a few realistic habits.', time: '5 min read', tint: 'yellow' },
  { icon: '↗', type: 'Career', title: 'Explore what feels right for you', text: 'Questions and resources to help you reflect on your interests and next steps.', time: '6 min read', tint: 'lavender' },
]
const counselors = [
  { name: 'Counselor Name', title: 'Registered Guidance Counselor', initials: 'CN', focus: 'College of Education · 1st–4th year', color: 'sage' },
  { name: 'Counselor Name', title: 'Guidance Associate', initials: 'CN', focus: 'College of Arts & Sciences · All levels', color: 'peach' },
  { name: 'Counselor Name', title: 'Career Guidance Counselor', initials: 'CN', focus: 'College of Agriculture · All levels', color: 'blue' },
]
const filteredAnnouncements = computed(() => {
  const query = announcementSearch.value.trim().toLowerCase()
  return announcements.filter((item) => (activeFilter.value === 'All' || item.category === activeFilter.value) &&
    (!query || `${item.title} ${item.text} ${item.tag}`.toLowerCase().includes(query)))
})
const filteredFaqs = computed(() => {
  const query = faqSearch.value.trim().toLowerCase()
  return faqs.filter((item) => !query || `${item.question} ${item.answer}`.toLowerCase().includes(query))
})
const today = new Date()
const minDate = `${today.getFullYear()}-${String(today.getMonth() + 1).padStart(2, '0')}-${String(today.getDate()).padStart(2, '0')}`

function goTo(item) {
  page.value = item
  mobileMenuOpen.value = false
  counselingMenuOpen.value = false
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function toggleTheme() {
  darkMode.value = !darkMode.value
  localStorage.setItem('compass-theme', darkMode.value ? 'dark' : 'light')
}

function submitBooking() {
  if (!booking.value.date || !booking.value.time) {
    bookingMessage.value = 'Choose a date and time to continue.'
    return
  }
  appointmentHistory.value.unshift({ ...booking.value, status: 'Pending', createdAt: new Date().toISOString() })
  localStorage.setItem('compass-appointments', JSON.stringify(appointmentHistory.value))
  bookingMessage.value = 'Your demo appointment request has been saved in this browser.'
  booking.value = { counselor: 'Any available counselor', date: '', time: '' }
}

function updateRequest(request, status) {
  request.status = status
}

function updateAppointment(index, status) {
  appointmentHistory.value[index].status = status
  localStorage.setItem('compass-appointments', JSON.stringify(appointmentHistory.value))
}

function submitConcern() {
  if (!concern.value.trim()) {
    concernMessage.value = 'Write a little about what is on your mind first.'
    return
  }
  const items = JSON.parse(localStorage.getItem('compass-concerns') || '[]')
  items.unshift({ text: concern.value.trim(), createdAt: new Date().toISOString() })
  localStorage.setItem('compass-concerns', JSON.stringify(items))
  concern.value = ''
  concernMessage.value = 'Thank you for sharing. This demo stores your message only in this browser and is not monitored.'
}

function submitMood() {
  if (!mood.value) {
    moodMessage.value = 'Choose the word that best describes today.'
    return
  }
  const checkins = JSON.parse(localStorage.getItem('compass-moods') || '[]')
  checkins.unshift({ mood: mood.value, day: new Date().toLocaleDateString() })
  moodHistory.value = checkins.slice(0, 7)
  localStorage.setItem('compass-moods', JSON.stringify(moodHistory.value))
  moodMessage.value = 'Check-in saved privately in this browser.'
}

function submitLogin() {
  if (!login.value.id.trim() || !login.value.password) {
    loginError.value = 'Enter your student ID or email and password.'
    loginSuccess.value = false
    return
  }
  loginError.value = ''
  loginSuccess.value = true
  page.value = 'Dashboard'
}

function openBooking() {
  bookingMessage.value = ''
  showBooking.value = true
}

function submitSurvey() {
  if (!surveyRating.value) {
    surveyMessage.value = 'Choose a rating before submitting.'
    return
  }
  const surveys = JSON.parse(localStorage.getItem('compass-surveys') || '[]')
  surveys.unshift({ rating: surveyRating.value, createdAt: new Date().toISOString() })
  localStorage.setItem('compass-surveys', JSON.stringify(surveys))
  surveyMessage.value = 'Thank you. Your demo feedback was saved in this browser.'
  surveyRating.value = ''
}
</script>

<template>
  <div class="site-shell" :class="{ 'dark-mode': darkMode }">
    <div class="topline">
      <div class="container topline-inner">
        <span><span class="top-icon">✉</span> guidance.sancarlos@cpsu.edu.ph <i></i> <span class="top-icon">⌕</span> (034) 312-0000 local 123 <b class="contact-placeholder">PLACEHOLDER CONTACTS — REPLACE BEFORE LAUNCH</b></span>
        <a href="https://www.facebook.com/" target="_blank" rel="noreferrer">Follow us on Facebook <span>↗</span></a>
      </div>
    </div>

    <header class="site-header">
      <div class="container nav-row">
        <button class="brand" type="button" aria-label="CPSU Compass home" @click="goTo('Home')">
          <span class="brand-mark"><span>✳</span></span>
          <span class="brand-copy"><strong>CPSU <b>COMPASS</b></strong><small>GUIDANCE &amp; COUNSELING CENTER</small></span>
        </button>
        <button class="mobile-menu-toggle" type="button" :aria-expanded="mobileMenuOpen" aria-label="Toggle navigation" @click="mobileMenuOpen = !mobileMenuOpen">☰</button>
        <nav class="main-nav" :class="{ 'nav-open': mobileMenuOpen }" aria-label="Main navigation">
          <template v-for="item in navigation" :key="item">
            <div v-if="item === 'Counseling'" class="nav-dropdown">
              <button type="button" :class="{ active: page === item }" :aria-expanded="counselingMenuOpen" @click="counselingMenuOpen = !counselingMenuOpen">Counseling<span class="nav-chevron">⌄</span></button>
              <div v-if="counselingMenuOpen" class="dropdown-menu"><button type="button" @click="goTo('Counseling')">Online Counseling</button><button type="button" @click="goTo('Counseling')">Office Counseling</button></div>
            </div>
            <button v-else type="button" :class="{ active: page === item }" @click="goTo(item)">{{ item }}</button>
          </template>
          <button class="notification-toggle" type="button" aria-label="Notifications" :aria-expanded="showNotifications" @click="showNotifications = !showNotifications">♢<span>2</span></button>
          <button class="theme-toggle" type="button" :aria-label="darkMode ? 'Switch to light mode' : 'Switch to dark mode'" @click="toggleTheme">{{ darkMode ? '☀' : '☾' }}</button>
        </nav>
        <div v-if="showNotifications" class="notification-panel" role="region" aria-label="Recent notifications">
          <b>Just for you</b>
          <p><span>✳</span> Wellness Week updates are on the noticeboard.</p>
          <p><span>♡</span> Appointment changes will appear here.</p>
          <small>Demo notifications · no live alerts are sent</small>
        </div>
      </div>
    </header>

    <main>
      <template v-if="page === 'Home'">
        <section class="hero-section">
          <div class="container hero-grid">
            <div class="hero-copy">
              <div class="eyebrow"><span class="eyebrow-dot"></span> A SPACE TO PAUSE, GROW &amp; BEGIN AGAIN</div>
              <h1>Find your way.<br /><em>We’re here</em> with you.</h1>
              <p class="hero-tagline">Grow with direction. Shine with purpose.</p>
              <p class="hero-description">College brings a lot of new things. You don’t have to figure them all out alone. Whatever’s on your mind, there’s a place for you here.</p>
              <div class="hero-actions">
                <button class="button button-primary" type="button" @click="openBooking">Schedule an appointment <span>↗</span></button>
                <button class="text-button" type="button" @click="goTo('About Us')">Get to know us <span>→</span></button>
              </div>
              <div class="hero-note"><span class="avatar-stack"><i>🌱</i><i>♡</i><i>✳</i></span><span>A little support can make a big difference.</span></div>
            </div>
            <div class="hero-art" aria-label="Illustration of a student finding their way">
              <div class="art-sun"></div><div class="art-hill hill-back"></div><div class="art-hill hill-front"></div>
              <div class="art-path"></div><span class="art-star star-one">✳</span><span class="art-star star-two">✦</span>
              <span class="art-label">YOUR JOURNEY,<br />YOUR OWN PACE</span>
              <div class="art-person"><span class="person-head"></span><span class="person-hair"></span><span class="person-body"></span><span class="person-book"></span></div>
              <div class="art-caption"><span class="caption-icon">↗</span><span><b>One step at a time.</b><small>And you’ve already begun.</small></span></div>
            </div>
          </div>
          <div class="hero-bottom container"><span>GUIDANCE, COUNSELING &amp; TESTING CENTER</span><span class="hero-location">CENTRAL PHILIPPINES STATE UNIVERSITY <i>·</i> SAN CARLOS CAMPUS</span></div>
        </section>

        <section class="welcome-strip">
          <div class="container welcome-inner"><span class="welcome-flower">✳</span><p><b>Whatever you’re carrying, you don’t have to carry it alone.</b> Talk with someone who’s here to listen—no judgment, just support.</p><button type="button" @click="goTo('Counseling')">Explore counseling <span>→</span></button></div>
        </section>

        <section class="section-block container services-section">
          <div class="section-heading"><div><div class="eyebrow">HOW WE CAN HELP</div><h2>Support for wherever<br />you are right now.</h2></div><p>Big feelings, little worries, or somewhere in between. We’re here for all of it.</p></div>
          <div class="service-grid">
            <button class="service-card" type="button" @click="goTo('Counseling')"><span class="service-icon green-icon">♡</span><span class="service-arrow">↗</span><h3>Talk it through</h3><p>One-on-one counseling that meets you with care, where you are.</p><span class="card-link">Explore counseling →</span></button>
            <button class="service-card" type="button" @click="goTo('Resources')"><span class="service-icon yellow-icon">✳</span><span class="service-arrow">↗</span><h3>Find your next step</h3><p>Guidance for your studies, career choices, and what comes next.</p><span class="card-link">Explore guidance →</span></button>
            <button class="service-card" type="button" @click="goTo('Forms')"><span class="service-icon peach-icon">⌑</span><span class="service-arrow">↗</span><h3>Get connected</h3><p>Request support for yourself or connect someone to the right help.</p><span class="card-link">View forms →</span></button>
          </div>
        </section>

        <section class="section-block news-section">
          <div class="container news-layout"><div class="news-intro"><div class="eyebrow">AROUND THE CENTER</div><h2>A little news<br />from our corner.</h2><p>Updates, opportunities, and things happening around campus.</p><button class="text-button" type="button" @click="goTo('Announcements')">All announcements <span>→</span></button></div>
            <div class="news-cards"><article v-for="item in announcements.slice(0, 2)" :key="item.title" class="news-card"><div class="news-meta"><span :class="item.category === 'Scholarship' ? 'tag-yellow' : 'tag-green'">{{ item.category }}</span><span>{{ item.date }}</span></div><h3>{{ item.title }}</h3><p>{{ item.text }}</p><button type="button" class="news-link" @click="goTo('Announcements')">Read more <span>→</span></button></article></div>
          </div>
        </section>

        <section class="section-block container counselor-section"><div class="section-heading"><div><div class="eyebrow">PEOPLE IN YOUR CORNER</div><h2>Meet your support team.</h2></div><button class="text-button" type="button" @click="goTo('About Us')">Meet the whole team <span>→</span></button></div>
          <div class="counselor-grid"><article v-for="(person, index) in counselors" :key="index" class="counselor-card"><div class="counselor-avatar" :class="person.color"><span>{{ person.initials }}</span><i>✳</i></div><div class="counselor-info"><h3>{{ person.name }}</h3><p>{{ person.title }}</p><small>{{ person.focus }}</small><a href="mailto:guidance.sancarlos@cpsu.edu.ph">Send an email ↗</a></div></article></div>
          <p class="placeholder-note">Counselor names and assignments are placeholders. Please confirm current staff details with the campus.</p>
        </section>

        <section class="concern-section"><div class="container concern-layout"><div><div class="eyebrow">NO PRESSURE. JUST A START.</div><h2>Not ready to talk<br />face to face?</h2><p>Leave a note about what’s on your mind. You can keep it anonymous. This demo box is not monitored by staff.</p></div><form class="concern-form" @submit.prevent="submitConcern"><label class="sr-only" for="concern-text">Share a concern anonymously</label><textarea id="concern-text" v-model="concern" placeholder="It’s okay if you’re not sure where to start…"></textarea><div class="concern-form-footer"><span>🔒 This demo saves locally in your browser only.</span><button class="button button-primary" type="submit">Share anonymously <span>→</span></button></div><p v-if="concernMessage" class="form-message" role="status">{{ concernMessage }}</p></form></div></section>

        <section class="emergency-strip"><div class="container emergency-inner"><span class="emergency-icon">♡</span><div><b>Need help right now?</b><p>If you’re in immediate danger, contact local emergency services or visit the nearest hospital.</p></div><button type="button" @click="showEmergency = true">See support options <span>→</span></button></div></section>
      </template>

      <template v-else-if="page === 'About Us'">
        <section class="page-hero"><div class="container"><div class="eyebrow">A LITTLE ABOUT US</div><h1>A place to feel heard,<br /><em>and find your way.</em></h1><p>We’re the Guidance, Counseling and Testing Center at CPSU San Carlos Campus. Our door is open to every student, at every step.</p></div></section>
        <section class="section-block container about-grid"><article class="about-panel"><span class="service-icon green-icon">✳</span><div class="eyebrow">OUR MISSION</div><h2>Here to help you thrive.</h2><p>To provide inclusive, student-centered guidance and counseling that supports personal growth, wellbeing, academic success, and informed decision-making.</p></article><article class="about-panel pale-panel"><span class="service-icon yellow-icon">◎</span><div class="eyebrow">OUR VISION</div><h2>A campus where everyone can flourish.</h2><p>A caring university community where every student can access the support and guidance they need to grow with confidence and purpose.</p></article></section>
        <section class="section-block about-details"><div class="container detail-grid"><div><div class="eyebrow">COME SAY HELLO</div><h2>Our door is open.</h2><p>Visit us at the CPSU San Carlos Campus Guidance Center. Walk-ins are welcome during office hours; counselor availability may vary.</p><div class="detail-line"><b>Office hours</b><span>Monday–Friday · 8:00 AM–5:00 PM<br />Please check campus announcements for updates.</span></div><div class="detail-line"><b>Find us</b><span>Central Philippines State University<br />San Carlos City, Negros Occidental</span></div><div class="detail-line"><b>Get in touch</b><span>guidance.sancarlos@cpsu.edu.ph<br />(034) 312-0000 local 123</span></div></div><div class="map-placeholder"><span>✳</span><b>CPSU SAN CARLOS CAMPUS</b><small>Map and office location placeholder</small><span class="map-pin">⌖</span></div></div></section>
      </template>

      <template v-else-if="page === 'Counseling'">
        <section class="page-hero"><div class="container"><div class="eyebrow">COUNSELING &amp; SUPPORT</div><h1>You don’t need<br />to have it <em>all figured out.</em></h1><p>Whether you need a listening ear or help sorting through a decision, we can take the next step together.</p><button class="button button-primary" type="button" @click="openBooking">Request an appointment <span>↗</span></button></div></section>
        <section class="section-block container counseling-options"><article class="option-card"><span class="service-icon green-icon">♡</span><div class="eyebrow">ONLINE COUNSELING</div><h2>Connect from wherever you are.</h2><p>Request an online session for a private, convenient way to talk. The center will confirm availability and the meeting details.</p><button class="text-button" type="button" @click="openBooking">Request online counseling <span>→</span></button></article><article class="option-card pale-panel"><span class="service-icon yellow-icon">⌖</span><div class="eyebrow">OFFICE COUNSELING</div><h2>A quiet space to talk.</h2><p>Meet with a counselor at the Guidance Center on campus. Walk-ins are welcome when a counselor is available.</p><button class="text-button" type="button" @click="openBooking">Request an office visit <span>→</span></button></article></section>
        <section class="section-block counselors-list"><div class="container"><div class="section-heading"><div><div class="eyebrow">YOUR SUPPORT TEAM</div><h2>Find a counselor.</h2></div></div><div class="counselor-grid"><article v-for="(person, index) in counselors" :key="index" class="counselor-card"><div class="counselor-avatar" :class="person.color"><span>{{ person.initials }}</span><i>✳</i></div><div class="counselor-info"><h3>{{ person.name }}</h3><p>{{ person.title }}</p><small>{{ person.focus }}</small><a href="mailto:guidance.sancarlos@cpsu.edu.ph">Send an email ↗</a></div></article></div><p class="placeholder-note">Staff details are placeholders. Replace with current counselor names, photos, and assignments.</p></div></section>
      </template>

      <template v-else-if="page === 'Forms'">
        <section class="page-hero"><div class="container"><div class="eyebrow">FORMS &amp; REQUESTS</div><h1>One small step<br />toward <em>what you need.</em></h1><p>Start a request here, or talk with the office if you’re not sure which form is right for you.</p></div></section>
        <section class="section-block container forms-grid"><article v-for="(form, index) in [{icon:'♡',name:'Counseling request',desc:'Ask for a confidential conversation with a counselor.',label:'Start a request'},{icon:'⌑',name:'Referral form',desc:'Connect a student with the Guidance Center for support.',label:'Request a referral'},{icon:'✳',name:'Good moral certificate',desc:'Review how to request a certificate through the campus.',label:'View requirements'},{icon:'↗',name:'Exit interview',desc:'Share your experience and complete an exit interview request.',label:'Start an interview'}]" :key="form.name" class="form-card"><span class="service-icon" :class="index % 2 ? 'yellow-icon' : 'green-icon'">{{ form.icon }}</span><h2>{{ form.name }}</h2><p>{{ form.desc }}</p><button class="text-button" type="button" @click="index === 0 ? openBooking() : bookingMessage = 'Demo only: add the approved campus form link here.'">{{ form.label }} <span>→</span></button></article><article class="form-card"><span class="service-icon yellow-icon">♡</span><h2>Session satisfaction survey</h2><p>Share feedback after a counseling session. Your demo response stays in this browser.</p><button class="text-button" type="button" @click="showSurvey = true; surveyMessage = ''">Open survey <span>→</span></button></article><p class="demo-banner">These are demo request links. Connect approved campus forms and document requirements before publishing.</p><p v-if="bookingMessage" class="form-message" role="status">{{ bookingMessage }}</p></section>
      </template>

      <template v-else-if="page === 'Resources'">
        <section class="page-hero"><div class="container"><div class="eyebrow">YOUR SELF-HELP CORNER</div><h1>Little things that<br />can make <em>a difference.</em></h1><p>Browse gentle, practical starting points for your wellbeing, studies, and future.</p></div></section>
        <section class="section-block container"><div class="section-heading"><div><div class="eyebrow">A FEW PLACES TO BEGIN</div><h2>Resources for real life.</h2></div></div><div class="resource-grid"><article v-for="item in resources" :key="item.title" class="resource-card"><span class="resource-art" :class="item.tint">{{ item.icon }}</span><div class="resource-content"><small>{{ item.type }} <i>·</i> {{ item.time }}</small><h3>{{ item.title }}</h3><p>{{ item.text }}</p><button class="text-button" type="button" @click="bookingMessage = 'This demo resource is a placeholder for reviewed campus content.'">Explore resource <span>→</span></button></div></article></div><div class="career-banner"><div><div class="eyebrow">CAREER &amp; COURSE GUIDANCE</div><h2>Curious about what comes next?</h2><p>Explore your interests, learn about aptitude testing, and talk through your options with a counselor.</p></div><button class="button button-primary" type="button" @click="openBooking">Talk to a counselor <span>→</span></button></div></section>
      </template>

      <template v-else-if="page === 'Announcements'">
        <section class="page-hero"><div class="container"><div class="eyebrow">WHAT’S HAPPENING</div><h1>A little something<br />for <em>your noticeboard.</em></h1><p>Campus updates, scholarships, and opportunities from the Guidance Center.</p></div></section>
        <section class="section-block container announcement-page"><div class="announcement-controls"><div class="filter-pills"><button v-for="filter in ['All','Announcement','Scholarship']" :key="filter" type="button" :class="{ selected: activeFilter === filter }" @click="activeFilter = filter">{{ filter }}<span v-if="filter === 'All'"> ({{ announcements.length }})</span></button></div><label class="search-box"><span>⌕</span><input v-model="announcementSearch" type="search" placeholder="Search updates…" aria-label="Search announcements" /></label></div><div v-if="filteredAnnouncements.length" class="announcement-list"><article v-for="item in filteredAnnouncements" :key="item.title" class="announcement-card"><div class="announcement-date"><b>{{ item.date.split(' ')[1].replace(',', '') }}</b><span>{{ item.date.split(' ')[0] }}</span></div><div class="announcement-copy"><div class="news-meta"><span :class="item.category === 'Scholarship' ? 'tag-yellow' : 'tag-green'">{{ item.category }}</span><span>{{ item.tag }}</span></div><h2>{{ item.title }}</h2><p>{{ item.text }}</p></div><span class="announcement-arrow">↗</span></article></div><p v-else class="empty-state">No updates match your search. Try another word or filter.</p><p class="placeholder-note">Demo announcements and scholarship notices. Please verify deadlines and details with the university.</p></section>
      </template>

      <template v-else-if="page === 'FAQ'">
        <section class="page-hero"><div class="container"><div class="eyebrow">GOOD QUESTIONS</div><h1>Wondering about<br /><em>something?</em></h1><p>Here are a few answers to help you feel more at ease before you reach out.</p></div></section>
        <section class="section-block container faq-layout"><div><label class="search-box faq-search"><span>⌕</span><input v-model="faqSearch" type="search" placeholder="Search a question…" aria-label="Search frequently asked questions" /></label><p class="faq-note">Can’t find what you’re looking for? <a href="mailto:guidance.sancarlos@cpsu.edu.ph">Get in touch with us →</a></p></div><div class="faq-list"><details v-for="item in filteredFaqs" :key="item.question"><summary>{{ item.question }}<span>+</span></summary><p>{{ item.answer }}</p></details><p v-if="!filteredFaqs.length" class="empty-state">No matching questions found.</p></div></section>
      </template>

      <template v-else-if="page === 'Login'">
        <section class="login-page"><div class="login-card"><div class="login-logo"><span class="brand-mark"><span>✳</span></span><div class="eyebrow">CPSU COMPASS</div></div><h1>Welcome back.</h1><p class="login-subtitle">A little support is always here for you.</p><form @submit.prevent="submitLogin"><label for="login-id">Student ID or email</label><input id="login-id" v-model="login.id" type="text" autocomplete="username" placeholder="e.g. 2025-00000 or you@email.com" /><label for="login-password">Password</label><div class="password-field"><input id="login-password" v-model="login.password" :type="showPassword ? 'text' : 'password'" autocomplete="current-password" placeholder="Enter your password" /><button type="button" :aria-label="showPassword ? 'Hide password' : 'Show password'" @click="showPassword = !showPassword">{{ showPassword ? 'Hide' : 'Show' }}</button></div><label for="role-select">I’m signing in as</label><select id="role-select" v-model="selectedRole"><option>Student</option><option>Counselor</option><option>Admin</option></select><div class="login-extras"><label class="remember-label"><input v-model="login.remember" type="checkbox" /> Remember me</label><button type="button" class="forgot-link" @click="loginError = 'Password reset is not available in this demo. Please contact the Guidance Center.'">Forgot password?</button></div><p v-if="loginError" class="form-error" role="alert">{{ loginError }}</p><button class="button button-primary login-submit" type="submit">Log in <span>→</span></button><p v-if="loginSuccess" class="form-message" role="status">Demo sign-in only. A real login requires secure server-side authentication.</p></form><div class="privacy-note"><span>♧</span><p>Your privacy matters. This demo does not authenticate users or connect to a real account. Real student data must be protected under the Data Privacy Act of 2012 (RA 10173).</p></div><small class="demo-login-hint">Demo: enter any ID/email and password to preview the {{ selectedRole.toLowerCase() }} dashboard.</small></div></section>
      </template>

      <template v-else-if="page === 'Dashboard'">
        <section class="dashboard-page"><div class="container"><div class="dashboard-welcome"><div><div class="eyebrow">{{ selectedRole.toUpperCase() }} DASHBOARD · DEMO</div><h1>Welcome, <em>{{ login.id.split('@')[0] || 'friend' }}.</em></h1><p>A gentle reminder: you can take things one step at a time.</p></div><button class="text-button" type="button" @click="goTo('Home')">Sign out <span>→</span></button></div>
          <template v-if="selectedRole === 'Student'">
            <div class="dashboard-grid">
              <article class="dashboard-card">
                <div class="eyebrow">YOUR NEXT APPOINTMENT</div>
                <h2>You’re in the right place.</h2>
                <p>When you request an appointment, you’ll find its status here.</p>
                <button class="button button-primary" type="button" @click="openBooking">Request an appointment <span>→</span></button>
              </article>
              <article class="dashboard-card">
                <div class="eyebrow">WEEKLY CHECK-IN</div>
                <h2>How are you feeling?</h2>
                <div class="mood-choices compact">
                  <button v-for="m in ['Rough','Okay','Good','Great']" :key="m" type="button" :class="{ chosen: mood === m }" @click="mood = m"><span>{{ { Rough: '☁', Okay: '◡', Good: '☀', Great: '✦' }[m] }}</span>{{ m }}</button>
                </div>
                <button class="text-button" type="button" @click="submitMood">Save check-in <span>→</span></button>
                <p v-if="moodMessage" class="form-message">{{ moodMessage }}</p>
                <div v-if="moodHistory.length" class="mood-chart" aria-label="Your weekly mood trend">
                  <span v-for="(entry, index) in moodHistory.slice().reverse()" :key="index" class="mood-bar" :style="{ height: { Rough: '25%', Okay: '48%', Good: '72%', Great: '94%' }[entry.mood] }"><i>{{ entry.day }}</i></span>
                </div>
                <p v-else class="mood-empty">Your private trend will appear here as you check in.</p>
                <small class="mood-privacy">Stored only in this browser · demo data</small>
              </article>
              <article class="dashboard-card wide-card">
                <div class="eyebrow">REQUEST HISTORY</div>
                <h2>Your requests</h2>
                <p v-if="!appointmentHistory.length">Your appointment requests will show up here.</p>
                <div v-for="(item, index) in appointmentHistory" :key="index" class="request-row">
                  <span><b>{{ item.counselor }}</b><small>{{ item.date }} · {{ item.time }}</small></span>
                  <em>{{ item.status }}</em>
                  <button v-if="item.status === 'Pending' || item.status === 'Approved'" class="cancel-request" type="button" @click="updateAppointment(index, 'Cancelled')">Cancel</button>
                </div>
              </article>
            </div>
          </template>
          <template v-else-if="selectedRole === 'Counselor'"><div class="dashboard-grid"><article class="dashboard-card"><div class="eyebrow">TODAY’S SCHEDULE</div><h2>A clear space for care.</h2><p>No sessions scheduled in this demo dashboard.</p><span class="status-pill">Demo schedule</span></article><article class="dashboard-card"><div class="eyebrow">APPOINTMENT REQUESTS</div><h2>Requests to review</h2><div v-for="(request, index) in counselorRequests" :key="index" class="request-row counselor-request"><span><b>{{ request.student }}</b><small>{{ request.detail }}</small></span><em>{{ request.status }}</em><div v-if="request.status === 'Pending'" class="request-actions"><button type="button" @click="updateRequest(request, 'Approved')">Approve</button><button type="button" @click="updateRequest(request, 'Rescheduled')">Reschedule</button><button type="button" @click="updateRequest(request, 'Declined')">Decline</button></div></div></article><article class="dashboard-card"><div class="eyebrow">ASSIGNED STUDENTS</div><h2>Your student support</h2><p>Assigned student information is not available in this demo.</p></article><article class="dashboard-card"><div class="eyebrow">PRIVATE SESSION NOTES</div><h2>Protected by design.</h2><p>Session notes must only be available in a secure, role-based production system.</p></article></div></template>
          <template v-else><div class="dashboard-grid"><article class="dashboard-card"><div class="eyebrow">CENTER MANAGEMENT</div><h2>Manage the center.</h2><p>Counselors, announcements, forms, and reports.</p><div class="admin-actions"><button type="button" @click="goTo('Announcements')">Announcements</button><button type="button" @click="goTo('Forms')">Forms</button><button type="button" @click="goTo('About Us')">Counselors</button></div></article><article class="dashboard-card"><div class="eyebrow">REPORTS</div><h2>Keep student data private.</h2><p>Connect reporting only after a secure, role-based backend is in place.</p><span class="status-pill">Demo data only</span></article><article class="dashboard-card wide-card"><div class="eyebrow">PRIVATE SESSION NOTES</div><h2>Protected by design.</h2><p>Real session notes must only be available in a secure, role-based production system.</p></article></div></template>
          <p class="demo-banner">This is a sample dashboard using local demo state. Do not enter real student information.</p></div></section>
      </template>
    </main>

    <footer class="site-footer"><div class="container footer-main"><div class="footer-brand"><button class="brand footer-logo" type="button" @click="goTo('Home')"><span class="brand-mark"><span>✳</span></span><span class="brand-copy"><strong>CPSU <b>COMPASS</b></strong><small>GUIDANCE &amp; COUNSELING CENTER</small></span></button><p>Grow with direction.<br />Shine with purpose.</p></div><div class="footer-col"><b>Find your way</b><button v-for="item in ['About Us','Counseling','Forms']" :key="item" type="button" @click="goTo(item)">{{ item }}</button></div><div class="footer-col"><b>Here for you</b><button v-for="item in ['Resources','Announcements','FAQ']" :key="item" type="button" @click="goTo(item)">{{ item }}</button></div><div class="footer-col footer-contact"><b>Say hello</b><a href="mailto:guidance.sancarlos@cpsu.edu.ph">guidance.sancarlos@cpsu.edu.ph</a><a href="tel:+63343120000">(034) 312-0000 local 123</a><a href="https://www.facebook.com/" target="_blank" rel="noreferrer">Facebook page ↗</a></div></div><div class="container footer-bottom"><span>© {{ new Date().getFullYear() }} CPSU-San Carlos Campus Guidance System</span><span>Student wellbeing comes first. <button type="button" @click="goTo('About Us')">Data Privacy</button></span><span>Made with care <b>✳</b></span></div></footer>

    <button class="emergency-fab" type="button" @click="showEmergency = true"><span>♡</span> Need help now?</button>

    <div v-if="showBooking" class="modal-backdrop" @click.self="showBooking = false" @keydown.esc="showBooking = false"><section class="modal-card" role="dialog" aria-modal="true" aria-labelledby="booking-title"><button class="modal-close" type="button" aria-label="Close appointment form" @click="showBooking = false">×</button><div class="eyebrow">A FIRST STEP</div><h2 id="booking-title">Request an appointment.</h2><p>Choose what works for you. The center will confirm a real appointment.</p><form @submit.prevent="submitBooking"><label for="booking-counselor">Counselor</label><select id="booking-counselor" v-model="booking.counselor"><option>Any available counselor</option><option>Counselor Name · Placeholder</option></select><div class="booking-fields"><div><label for="booking-date">Preferred date</label><input id="booking-date" v-model="booking.date" type="date" :min="minDate" required /></div><div><label for="booking-time">Available time</label><select id="booking-time" v-model="booking.time" required><option value="" disabled>Select a time</option><option>9:00 AM</option><option>10:30 AM</option><option>1:00 PM</option><option>2:30 PM</option></select></div></div><p class="privacy-inline">Demo only: no email or SMS is sent. Your request is saved in this browser.</p><button class="button button-primary login-submit" type="submit">Send request <span>→</span></button><p v-if="bookingMessage" class="form-message" role="status">{{ bookingMessage }}</p></form></section></div>

    <div v-if="showSurvey" class="modal-backdrop" @click.self="showSurvey = false" @keydown.esc="showSurvey = false"><section class="modal-card" role="dialog" aria-modal="true" aria-labelledby="survey-title"><button class="modal-close" type="button" aria-label="Close satisfaction survey" @click="showSurvey = false">×</button><div class="eyebrow">YOUR FEEDBACK MATTERS</div><h2 id="survey-title">How was your session?</h2><p>Your answers help the center improve. Please do not include sensitive details in this demo survey.</p><form @submit.prevent="submitSurvey"><label for="survey-rating">How supported did you feel?</label><select id="survey-rating" v-model="surveyRating" required><option value="" disabled>Choose a rating</option><option>Very supported</option><option>Supported</option><option>Somewhat supported</option><option>Not supported</option></select><button class="button button-primary login-submit" type="submit">Send feedback <span>→</span></button><p v-if="surveyMessage" class="form-message" role="status">{{ surveyMessage }}</p></form></section></div>

    <div v-if="showEmergency" class="modal-backdrop" @click.self="showEmergency = false" @keydown.esc="showEmergency = false"><section class="modal-card emergency-modal" role="dialog" aria-modal="true" aria-labelledby="emergency-title"><button class="modal-close" type="button" aria-label="Close support options" @click="showEmergency = false">×</button><span class="service-icon peach-icon">♡</span><div class="eyebrow">YOU DESERVE SUPPORT</div><h2 id="emergency-title">Let’s find help right now.</h2><p>If you’re in immediate danger or may hurt yourself or someone else, call local emergency services or go to the nearest emergency room. Don’t wait for this website or a message reply.</p><a class="emergency-call" href="tel:911">Call 911 <span>↗</span></a><div class="detail-line"><b>On campus</b><span>Visit the Guidance Center during office hours or contact campus security for urgent assistance.</span></div><div class="detail-line"><b>Need someone to listen?</b><span>Contact a trusted person and ask them to stay with you while you find support.</span></div><p class="placeholder-note">Confirm local hotline and campus emergency contact details before publishing.</p></section></div>
  </div>
</template>
