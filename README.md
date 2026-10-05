const STORAGE_KEY = 'callblock_blacklist_v1';
const THEME_KEY = 'callblock_theme_v1';

const DEFAULT_CONTACTS = [
  { name: 'Banco Central', phone: '+34600123456' },
  { name: 'Seguros Vida', phone: '+34600234567' },
  { name: 'Operador Soporte', phone: '+34600345678' },
  { name: 'Agencia de Viajes', phone: '+34600456789' },
  { name: 'Médico', phone: '+34600567890' },
  { name: 'Peluquería', phone: '+34600678901' }
];

const DEFAULT_RULES = [
  {
    id: crypto.randomUUID(),
    name: 'Banco Central',
    phone: '+34600123456',
    matchType: 'exact',
    reason: 'Número recurrente',
    createdAt: Date.now()
  },
  {
    id: crypto.randomUUID(),
    name: 'Cobro sospechoso',
    phone: '600',
    matchType: 'startsWith',
    reason: 'Prefijo sospechoso',
    createdAt: Date.now() + 1
  },
  {
    id: crypto.randomUUID(),
    name: 'Spam comercial',
    phone: '1234',
    matchType: 'contains',
    reason: 'Coincidencia comercial',
    createdAt: Date.now() + 2
  }
];

const elements = {
  form: document.querySelector('#blacklistForm'),
  nameInput: document.querySelector('#nameInput'),
  phoneInput: document.querySelector('#phoneInput'),
  matchType: document.querySelector('#matchType'),
  reasonInput: document.querySelector('#reasonInput'),
  contactList: document.querySelector('#contactList'),
  blacklistList: document.querySelector('#blacklistList'),
  searchInput: document.querySelector('#searchInput'),
  testInput: document.querySelector('#testInput'),
  checkNumberBtn: document.querySelector('#checkNumberBtn'),
  checkResult: document.querySelector('#checkResult'),
  themeToggle: document.querySelector('#themeToggle'),
  statTotal: document.querySelector('#statTotal'),
  statExact: document.querySelector('#statExact'),
  statStartsWith: document.querySelector('#statStartsWith'),
  statContains: document.querySelector('#statContains')
};

let blacklist = loadBlacklist();
let contacts = loadContacts();

initialize();

function initialize() {
  applyTheme(loadTheme());
  renderContacts();
  renderBlacklist();
  bindEvents();
}

function bindEvents() {
  elements.form.addEventListener('submit', handleAddRule);
  elements.checkNumberBtn.addEventListener('click', handleCheckNumber);
  elements.searchInput.addEventListener('input', renderBlacklist);
  elements.themeToggle.addEventListener('click', toggleTheme);
}

function loadBlacklist() {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (!saved) {
    return DEFAULT_RULES;
  }

  try {
    const parsed = JSON.parse(saved);
    return Array.isArray(parsed) && parsed.length ? parsed : DEFAULT_RULES;
  } catch {
    return DEFAULT_RULES;
  }
}

function loadContacts() {
  const saved = localStorage.getItem('callblock_contacts_v1');
  if (!saved) {
    return DEFAULT_CONTACTS;
  }

  try {
    const parsed = JSON.parse(saved);
    return Array.isArray(parsed) && parsed.length ? parsed : DEFAULT_CONTACTS;
  } catch {
    return DEFAULT_CONTACTS;
  }
}

function loadTheme() {
  return localStorage.getItem(THEME_KEY) || 'dark';
}

function saveBlacklist() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(blacklist));
}

function saveContacts() {
  localStorage.setItem('callblock_contacts_v1', JSON.stringify(contacts));
}

function applyTheme(mode) {
  document.body.classList.toggle('light', mode === 'light');
  elements.themeToggle.textContent = mode === 'light' ? '🌙' : '☀️';
  localStorage.setItem(THEME_KEY, mode);
}

function toggleTheme() {
  const isLight = document.body.classList.contains('light');
  applyTheme(isLight ? 'dark' : 'light');
}

function handleAddRule(event) {
  event.preventDefault();

  const name = elements.nameInput.value.trim();
  const phone = elements.phoneInput.value.trim();
  const matchType = elements.matchType.value;
  const reason = elements.reasonInput.value.trim() || 'Número bloqueado';

  if (!name || !phone) {
    alert('Debes completar nombre y número.');
    return;
  }

  const newRule = {
    id: crypto.randomUUID(),
    name,
    phone: phone,
    matchType,
    reason,
    createdAt: Date.now()
  };

  blacklist.unshift(newRule);
  saveBlacklist();

  elements.form.reset();
  renderBlacklist();
  renderStats();
}

function handleCheckNumber() {
  const candidate = elements.testInput.value.trim();
  if (!candidate) {
    elements.checkResult.textContent = 'Introduce un número para comprobar si está bloqueado.';
    elements.checkResult.className = 'result-box neutral';
    return;
  }

  const match = findMatchingRule(candidate);

  if (!match) {
    elements.checkResult.innerHTML = `
      <strong>Resultado:</strong> el número <strong>${candidate}</strong> no coincide con ninguna regla de la lista negra.
    `;
    elements.checkResult.className = 'result-box neutral';
    return;
  }

  const summary = `
    <strong>Bloqueado:</strong> sí.<br>
    <strong>Número:</strong> ${candidate}<br>
    <strong>Coincidencia:</strong> ${match.name} (${labelForMatch(match.matchType)})<br>
    <strong>Motivo:</strong> ${match.reason}
  `;

  elements.checkResult.innerHTML = summary;
  elements.checkResult.className = 'result-box negative';
}

function findMatchingRule(candidate) {
  const normalizedCandidate = toComparable(candidate);

  return blacklist.find((rule) => {
    const normalizedRule = toComparable(rule.phone);
    if (!normalizedRule || !normalizedCandidate) return false;

    if (rule.matchType === 'exact') return normalizedCandidate === normalizedRule;
    if (rule.matchType === 'startsWith') return normalizedCandidate.startsWith(normalizedRule);
    if (rule.matchType === 'contains') return normalizedCandidate.includes(normalizedRule);
    return false;
  });
}

function renderContacts() {
  elements.contactList.innerHTML = '';

  contacts.forEach((contact) => {
    const button = document.createElement('button');
    button.type = 'button';
    button.className = 'contact-chip';
    button.textContent = `${contact.name} • ${contact.phone}`;
    button.addEventListener('click', () => {
      elements.nameInput.value = contact.name;
      elements.phoneInput.value = contact.phone;
      elements.matchType.value = 'exact';
      elements.reasonInput.value = 'Contacto bloqueado';
      elements.nameInput.focus();
    });
    elements.contactList.appendChild(button);
  });
}

function renderBlacklist() {
  const phrase = elements.searchInput.value.trim().toLowerCase();
  const filtered = blacklist.filter((rule) => {
    const haystack = `${rule.name} ${rule.phone} ${rule.reason}`.toLowerCase();
    return haystack.includes(phrase);
  });

  if (!filtered.length) {
    elements.blacklistList.innerHTML = '<li class="empty-state">No se encontraron coincidencias.</li>';
    renderStats();
    return;
  }

  elements.blacklistList.innerHTML = filtered
    .map(
      (rule) => `
        <li class="blacklist-item">
          <div class="blacklist-head">
            <span class="blacklist-name">${escapeHTML(rule.name)}</span>
            <span class="blacklist-number">${escapeHTML(rule.phone)}</span>
          </div>
          <span class="match-badge">${labelForMatch(rule.matchType)}</span>
          <div class="action-cell">
            <button class="delete-btn" data-id="${rule.id}" type="button">Eliminar</button>
          </div>
        </li>
      `
    )
    .join('');

  elements.blacklistList.querySelectorAll('.delete-btn').forEach((button) => {
    button.addEventListener('click', () => {
      blacklist = blacklist.filter((rule) => rule.id !== button.dataset.id);
      saveBlacklist();
      renderBlacklist();
      renderStats();
    });
  });

  renderStats();
}

function renderStats() {
  const total = blacklist.length;
  const exact = blacklist.filter((rule) => rule.matchType === 'exact').length;
  const startsWith = blacklist.filter((rule) => rule.matchType === 'startsWith').length;
  const contains = blacklist.filter((rule) => rule.matchType === 'contains').length;

  elements.statTotal.textContent = total;
  elements.statExact.textContent = exact;
  elements.statStartsWith.textContent = startsWith;
  elements.statContains.textContent = contains;
}

function labelForMatch(type) {
  if (type === 'exact') return 'Exacto';
  if (type === 'startsWith') return 'Empieza por';
  if (type === 'contains') return 'Contiene';
  return 'Otro';
}

function toComparable(value) {
  if (!value) return '';
  return value
    .toString()
    .trim()
    .toLowerCase()
    .replace(/\s+/g, '')
    .replace(/[^\d+]/g, '');
}

function escapeHTML(value) {
  return value
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/\"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

window.addEventListener('beforeunload', () => {
  saveBlacklist();
  saveContacts();
});
