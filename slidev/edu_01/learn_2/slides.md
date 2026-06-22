---
theme: ./theme
title: Редизайн мобильного банкинга — UX-исследование
colorSchema: light
layout: none
fonts:
  sans: Manrope
  serif: DM Sans
  mono: JetBrains Mono
transition: fade
aspectRatio: 16/9
---

<div class="s1-bg gradient-mesh-1"></div>
<div class="s1-content">
  <div class="s1-eyebrow">ОТДЕЛ ЦИФРОВЫХ ПРОДУКТОВ · FINTRUST</div>
  <h1 class="s1-heading">Мобильный банк нового поколения</h1>
  <p class="s1-sub">UX-исследование и дизайн-концепция</p>
</div>
<div class="slide-footer slide-footer-dark">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span style="margin-left:auto;">1 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s1-bg { position: absolute; inset: 0; z-index: 0; }
.s1-content { position: absolute; inset: 0; z-index: 1; display: flex; flex-direction: column; justify-content: center; padding: 64px 72px 80px; }
.s1-eyebrow { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: rgba(255,255,255,0.5); margin-bottom: 20px; display: block; }
.s1-heading { font-family: var(--font-heading); font-size: 5rem; font-weight: 800; color: #FFFFFF; line-height: 1.05; margin: 0 0 24px; max-width: 65%; }
.s1-sub { font-family: var(--font-body); font-size: 1.35rem; color: rgba(255,255,255,0.65); margin: 0; }
</style>

---
layout: none
---

<div class="s2-root">
  <div class="s2-left">
    <span class="s2-eyebrow">ПОВЕСТКА</span>
    <h2 class="s2-heading">Что обсудим сегодня</h2>
  </div>
  <div class="s2-right">
    <span class="s2-num">01</span><span class="s2-text">Проблемы текущего приложения</span>
    <span class="s2-line"></span>
    <span class="s2-num">02</span><span class="s2-text">Исследование пользователей</span>
    <span class="s2-line"></span>
    <span class="s2-num">03</span><span class="s2-text">Дизайн-концепции</span>
    <span class="s2-line"></span>
    <span class="s2-num">04</span><span class="s2-text">План внедрения</span>
  </div>
</div>
<div class="slide-footer">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span style="margin-left:auto;">2 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s2-root { position: absolute; inset: 0; display: flex; padding-bottom: 44px; background: var(--bg-base); }
.s2-left { width: 42%; display: flex; flex-direction: column; justify-content: center; padding: 56px 48px 56px 64px; }
.s2-eyebrow { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: var(--color-muted); margin-bottom: 16px; display: block; }
.s2-heading { font-family: var(--font-heading); font-size: 5rem; font-weight: 800; color: var(--color-text); line-height: 1.05; margin: 0; }
.s2-right { flex: 1; display: grid; grid-template-columns: 40px 1fr; align-content: center; padding: 56px 64px; gap: 0; row-gap: 0; }
.s2-num { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: var(--color-muted); padding: 18px 0; align-self: center; }
.s2-text { font-family: var(--font-body); font-size: 1.35rem; font-weight: 500; color: var(--color-text); line-height: 1.3; padding: 18px 0; }
.s2-line { grid-column: 1 / -1; height: 1px; background: var(--color-surface-border); }
</style>

---
layout: none
---

<div class="s3-bg gradient-mesh-2"></div>
<div class="s3-content">
  <span class="s3-eyebrow">РАЗДЕЛ 01</span>
  <h2 class="s3-heading">Проблемы</h2>
  <p class="s3-sub">Что не работает сегодня</p>
</div>
<div class="slide-footer slide-footer-dark">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span>Проблемы</span>
  <span style="margin-left:auto;">3 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s3-bg { position: absolute; inset: 0; z-index: 0; }
.s3-content { position: absolute; inset: 0; z-index: 1; display: flex; flex-direction: column; justify-content: center; padding: 64px 72px 80px; }
.s3-eyebrow { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: rgba(255,255,255,0.45); margin-bottom: 16px; display: block; }
.s3-heading { font-family: var(--font-heading); font-size: 5.5rem; font-weight: 800; color: #FFFFFF; line-height: 1; margin: 0 0 20px; }
.s3-sub { font-family: var(--font-body); font-size: 1.35rem; color: rgba(255,255,255,0.6); margin: 0; }
</style>

---
layout: none
---

<div class="s4-root">
  <div class="s4-left">
    <h2 class="s4-heading">73% пользователей не могут найти перевод за 3 тапа</h2>
  </div>
  <div class="s4-right">
    <span class="s4-stat">4 года</span>
    <span class="s4-unit">без рефакторинга навигации</span>
    <p class="s4-desc">Навигация усложнялась без рефакторинга. Глубина меню достигла 5 уровней в платёжном разделе.</p>
    <div class="s4-compare">
      <span class="s4-tag">Целевая глубина: 3</span>
      <span class="s4-arrow">→</span>
      <span class="s4-current">Текущая: 5 уровней</span>
    </div>
  </div>
</div>
<div class="slide-footer">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span>Проблемы</span>
  <span style="margin-left:auto;">4 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s4-root { position: absolute; inset: 0; display: flex; padding-bottom: 44px; background: var(--bg-base); }
.s4-left { width: 50%; display: flex; flex-direction: column; justify-content: center; padding: 56px 48px 56px 64px; }
.s4-heading { font-family: var(--font-heading); font-size: 2.6rem; font-weight: 800; color: var(--color-text); line-height: 1.12; margin: 0; }
.s4-right { flex: 1; display: flex; flex-direction: column; justify-content: center; padding: 56px 64px; gap: 8px; }
.s4-stat { font-family: var(--font-heading); font-size: 4.5rem; font-weight: 800; color: var(--color-text); line-height: 1; display: block; }
.s4-unit { font-family: var(--font-heading); font-size: 1.25rem; font-weight: 600; color: var(--color-muted); display: block; margin-bottom: 8px; }
.s4-desc { font-family: var(--font-body); font-size: 1.2rem; color: var(--color-muted); line-height: 1.45; margin: 0 0 20px; }
.s4-compare { display: flex; align-items: center; gap: 12px; }
.s4-tag { font-family: var(--font-heading); font-size: 1rem; font-weight: 600; color: var(--color-muted); }
.s4-arrow { font-family: var(--font-heading); font-size: 1.25rem; color: var(--color-muted); }
.s4-current { font-family: var(--font-heading); font-size: 1rem; font-weight: 800; color: var(--color-text); }
</style>

---
layout: none
---

<div class="s5-root">
  <h2 class="s5-heading">Три критические точки боли</h2>
  <div class="s5-row">
    <div class="s5-meta">
      <span class="s5-num">01</span>
      <span class="s5-name">Авторизация</span>
      <span class="s5-kpi">40% отказов</span>
    </div>
    <div class="s5-meta">
      <span class="s5-num">02</span>
      <span class="s5-name">Переводы</span>
      <span class="s5-kpi">47 сек / операция</span>
    </div>
    <div class="s5-meta">
      <span class="s5-num">03</span>
      <span class="s5-name">Уведомления</span>
      <span class="s5-kpi">68% отключают push</span>
    </div>
  </div>
  <div class="s5-cards">
    <div class="s5-card card-solid"></div>
    <div class="s5-card card-solid"></div>
    <div class="s5-card card-solid"></div>
  </div>
</div>
<div class="slide-footer">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span>Проблемы</span>
  <span style="margin-left:auto;">5 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s5-root { position: absolute; inset: 0; display: flex; flex-direction: column; background: var(--bg-base); padding: 48px 64px 60px; }
.s5-heading { font-family: var(--font-heading); font-size: 2.4rem; font-weight: 800; color: var(--color-text); margin: 0 0 28px; line-height: 1.15; }
.s5-row { display: flex; gap: 20px; margin-bottom: 20px; }
.s5-meta { flex: 1; display: flex; flex-direction: column; gap: 4px; }
.s5-num { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: var(--color-muted); }
.s5-name { font-family: var(--font-heading); font-size: 1.2rem; font-weight: 700; color: var(--color-text); }
.s5-kpi { font-family: var(--font-heading); font-size: 1.5rem; font-weight: 800; color: var(--color-text); }
.s5-cards { display: flex; gap: 20px; flex: 1; }
.s5-card { flex: 1; border-radius: 13px; min-height: 0; }
</style>

---
layout: none
---

<div class="s6-bg gradient-mesh-3"></div>
<div class="s6-content">
  <span class="s6-eyebrow">РАЗДЕЛ 02</span>
  <h2 class="s6-heading">Исследование</h2>
  <p class="s6-sub">Что говорят пользователи</p>
</div>
<div class="slide-footer slide-footer-dark">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span>Исследование</span>
  <span style="margin-left:auto;">6 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s6-bg { position: absolute; inset: 0; z-index: 0; }
.s6-content { position: absolute; inset: 0; z-index: 1; display: flex; flex-direction: column; justify-content: center; padding: 64px 72px 80px; }
.s6-eyebrow { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: rgba(255,255,255,0.45); margin-bottom: 16px; display: block; }
.s6-heading { font-family: var(--font-heading); font-size: 5.5rem; font-weight: 800; color: #FFFFFF; line-height: 1; margin: 0 0 20px; }
.s6-sub { font-family: var(--font-body); font-size: 1.35rem; color: rgba(255,255,255,0.6); margin: 0; }
</style>

---
layout: none
---

<div class="s7-root">
  <div class="s7-left">
    <h2 class="s7-heading">Биометрия сократит время входа с 12 до 2 секунд</h2>
  </div>
  <div class="s7-right">
    <span class="s7-stat">89%</span>
    <span class="s7-unit">готовы использовать Face ID</span>
    <p class="s7-desc">Снижение отказов на входе с 40% до 8% — пятикратное улучшение конверсии авторизации.</p>
    <div class="s7-compare">
      <span class="s7-before">12 сек</span>
      <span class="s7-arrow">→</span>
      <span class="s7-after">2 сек</span>
    </div>
  </div>
</div>
<div class="slide-footer">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span>Исследование</span>
  <span style="margin-left:auto;">7 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s7-root { position: absolute; inset: 0; display: flex; padding-bottom: 44px; background: var(--bg-base); }
.s7-left { width: 50%; display: flex; flex-direction: column; justify-content: center; padding: 56px 48px 56px 64px; }
.s7-heading { font-family: var(--font-heading); font-size: 2.6rem; font-weight: 800; color: var(--color-text); line-height: 1.12; margin: 0; }
.s7-right { flex: 1; display: flex; flex-direction: column; justify-content: center; padding: 56px 64px; gap: 8px; }
.s7-stat { font-family: var(--font-heading); font-size: 5rem; font-weight: 800; color: var(--color-text); line-height: 1; display: block; }
.s7-unit { font-family: var(--font-heading); font-size: 1.25rem; font-weight: 600; color: var(--color-muted); display: block; margin-bottom: 4px; }
.s7-desc { font-family: var(--font-body); font-size: 1.2rem; color: var(--color-muted); line-height: 1.45; margin: 8px 0 16px; }
.s7-compare { display: flex; align-items: center; gap: 16px; }
.s7-before { font-family: var(--font-heading); font-size: 2rem; font-weight: 700; color: var(--color-muted); text-decoration: line-through; }
.s7-arrow { font-family: var(--font-heading); font-size: 1.5rem; color: var(--color-muted); }
.s7-after { font-family: var(--font-heading); font-size: 2rem; font-weight: 800; color: var(--color-text); }
</style>

---
layout: none
---

<div class="s8-root">
  <div class="s8-image card-solid"></div>
  <div class="s8-text">
    <h2 class="s8-heading">Карта пользовательского пути</h2>
    <p class="s8-desc">Визуализация текущего vs предлагаемого сценария перевода. Текущий путь — 7 экранов, 47 секунд. Целевой — 2 экрана, 15 секунд.</p>
    <div class="s8-badges">
      <span class="s8-badge">Сейчас: 7 шагов</span>
      <span class="s8-badge s8-badge-target">Цель: 2 шага</span>
    </div>
  </div>
</div>
<div class="slide-footer">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span>Исследование</span>
  <span style="margin-left:auto;">8 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s8-root { position: absolute; inset: 0; display: flex; align-items: stretch; padding: 44px 64px 60px; gap: 32px; background: var(--bg-base); }
.s8-image { flex: 1.3; border-radius: 13px; min-height: 100%; }
.s8-text { flex: 1; display: flex; flex-direction: column; justify-content: center; }
.s8-heading { font-family: var(--font-heading); font-size: 2.2rem; font-weight: 700; color: var(--color-text); line-height: 1.15; margin: 0 0 16px; }
.s8-desc { font-family: var(--font-body); font-size: 1.2rem; color: var(--color-muted); line-height: 1.5; margin: 0 0 24px; }
.s8-badges { display: flex; flex-direction: column; gap: 10px; }
.s8-badge { display: inline-block; font-family: var(--font-heading); font-size: 0.85rem; font-weight: 700; color: var(--color-muted); background: var(--color-surface); border-radius: 20px; padding: 8px 20px; line-height: 1; width: fit-content; }
.s8-badge-target { color: var(--color-text); background: var(--color-accent-bg); }
</style>

---
layout: none
---

<div class="s9-bg gradient-mesh-4"></div>
<div class="s9-content">
  <span class="s9-eyebrow">РАЗДЕЛ 03</span>
  <h2 class="s9-heading">Дизайн-концепции</h2>
  <p class="s9-sub">Как это будет выглядеть</p>
</div>
<div class="slide-footer slide-footer-dark">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span>Дизайн-концепции</span>
  <span style="margin-left:auto;">9 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s9-bg { position: absolute; inset: 0; z-index: 0; }
.s9-content { position: absolute; inset: 0; z-index: 1; display: flex; flex-direction: column; justify-content: center; padding: 64px 72px 80px; }
.s9-eyebrow { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: rgba(255,255,255,0.45); margin-bottom: 16px; display: block; }
.s9-heading { font-family: var(--font-heading); font-size: 5.5rem; font-weight: 800; color: #FFFFFF; line-height: 1; margin: 0 0 20px; }
.s9-sub { font-family: var(--font-body); font-size: 1.35rem; color: rgba(255,255,255,0.6); margin: 0; }
</style>

---
layout: none
---

<div class="s10-root">
  <div class="s10-left">
    <h2 class="s10-heading">Единый экран для 80% операций</h2>
  </div>
  <div class="s10-right">
    <span class="s10-stat">2 тапа</span>
    <span class="s10-unit">вместо 5 для любой операции</span>
    <p class="s10-desc">Концепция One-Screen Banking: перевод, оплата, история на одном экране. Снижение глубины навигации с 5 до 2 тапов.</p>
    <div class="s10-compare">
      <span class="s10-before">5 тапов</span>
      <span class="s10-arrow">→</span>
      <span class="s10-after">2 тапа</span>
    </div>
  </div>
</div>
<div class="slide-footer">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span>Дизайн-концепции</span>
  <span style="margin-left:auto;">10 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s10-root { position: absolute; inset: 0; display: flex; padding-bottom: 44px; background: var(--bg-base); }
.s10-left { width: 50%; display: flex; flex-direction: column; justify-content: center; padding: 56px 48px 56px 64px; }
.s10-heading { font-family: var(--font-heading); font-size: 2.6rem; font-weight: 800; color: var(--color-text); line-height: 1.12; margin: 0; }
.s10-right { flex: 1; display: flex; flex-direction: column; justify-content: center; padding: 56px 64px; gap: 8px; }
.s10-stat { font-family: var(--font-heading); font-size: 5rem; font-weight: 800; color: var(--color-text); line-height: 1; display: block; }
.s10-unit { font-family: var(--font-heading); font-size: 1.25rem; font-weight: 600; color: var(--color-muted); display: block; margin-bottom: 4px; }
.s10-desc { font-family: var(--font-body); font-size: 1.2rem; color: var(--color-muted); line-height: 1.45; margin: 8px 0 16px; }
.s10-compare { display: flex; align-items: center; gap: 16px; }
.s10-before { font-family: var(--font-heading); font-size: 2rem; font-weight: 700; color: var(--color-muted); text-decoration: line-through; }
.s10-arrow { font-family: var(--font-heading); font-size: 1.5rem; color: var(--color-muted); }
.s10-after { font-family: var(--font-heading); font-size: 2rem; font-weight: 800; color: var(--color-text); }
</style>

---
layout: none
---

<div class="s11-root">
  <h2 class="s11-heading">Три варианта интерфейса</h2>
  <div class="s11-row">
    <div class="s11-meta">
      <span class="s11-num">01</span>
      <span class="s11-name">Минималистичный</span>
      <span class="s11-desc">Фокус на 3 действиях</span>
    </div>
    <div class="s11-meta">
      <span class="s11-num">02</span>
      <span class="s11-name">Модульный</span>
      <span class="s11-desc">Виджеты по приоритету</span>
    </div>
    <div class="s11-meta">
      <span class="s11-num">03</span>
      <span class="s11-name">Адаптивный</span>
      <span class="s11-desc">ML-персонализация порядка</span>
    </div>
  </div>
  <div class="s11-cards">
    <div class="s11-card card-solid"></div>
    <div class="s11-card card-solid"></div>
    <div class="s11-card card-solid"></div>
  </div>
</div>
<div class="slide-footer">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span>Дизайн-концепции</span>
  <span style="margin-left:auto;">11 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s11-root { position: absolute; inset: 0; display: flex; flex-direction: column; background: var(--bg-base); padding: 48px 64px 60px; }
.s11-heading { font-family: var(--font-heading); font-size: 2.4rem; font-weight: 800; color: var(--color-text); margin: 0 0 24px; line-height: 1.15; }
.s11-row { display: flex; gap: 20px; margin-bottom: 20px; }
.s11-meta { flex: 1; display: flex; flex-direction: column; gap: 4px; }
.s11-num { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: var(--color-muted); }
.s11-name { font-family: var(--font-heading); font-size: 1.2rem; font-weight: 700; color: var(--color-text); }
.s11-desc { font-family: var(--font-body); font-size: 1.1rem; color: var(--color-muted); line-height: 1.35; }
.s11-cards { display: flex; gap: 20px; flex: 1; }
.s11-card { flex: 1; border-radius: 13px; min-height: 0; }
</style>

---
layout: none
---

<div class="s12-bg gradient-mesh-1"></div>
<div class="s12-content">
  <span class="s12-eyebrow">ПЛАН ВНЕДРЕНИЯ</span>
  <h2 class="s12-heading">Запуск бета-версии через 8 недель</h2>
  <p class="s12-sub">Прототип (2 нед) → Тестирование (3 нед) → Разработка (3 нед)</p>
  <span class="s12-contact-label">СВЯЖИТЕСЬ С КОМАНДОЙ</span>
  <span class="s12-contact">ux@fintrust.ru</span>
</div>
<div class="slide-footer slide-footer-dark">
  <span>FinTrust · Редизайн мобильного банкинга</span>
  <span style="margin-left:auto;">12 / 12</span>
</div>

<style>
.slidev-layout { padding: 0 !important; overflow: hidden; }
.s12-bg { position: absolute; inset: 0; z-index: 0; }
.s12-content { position: absolute; inset: 0; z-index: 1; display: flex; flex-direction: column; justify-content: center; padding: 64px 72px 80px; }
.s12-eyebrow { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: rgba(255,255,255,0.45); margin-bottom: 20px; display: block; }
.s12-heading { font-family: var(--font-heading); font-size: 3.8rem; font-weight: 800; color: #FFFFFF; line-height: 1.08; margin: 0 0 20px; max-width: 65%; }
.s12-sub { font-family: var(--font-body); font-size: 1.35rem; color: rgba(255,255,255,0.65); margin: 0 0 40px; }
.s12-contact-label { font-family: var(--font-heading); font-size: 0.7rem; font-weight: 600; text-transform: uppercase; letter-spacing: 0.15em; color: rgba(255,255,255,0.35); margin-bottom: 8px; display: block; }
.s12-contact { font-family: var(--font-body); font-size: 1.5rem; font-weight: 500; color: #FFFFFF; display: block; }
</style>
