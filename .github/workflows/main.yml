const fs = require('fs/promises');

// ============================================================
// قائمة السورسات (تم حذف الروابط المكررة لتسريع العملية)
// ============================================================
const sourceUrls = [
  "https://github.com/rtwo2/FastNodes/raw/refs/heads/main/sub/ipv4_only.txt",
  "https://github.com/crackbest/V2ray-Config/raw/refs/heads/main/config.txt",
  "https://github.com/Delta-Kronecker/V2ray-Config/raw/refs/heads/main/config/all_configs.txt",
  "https://github.com/10Dream/sub-mod/raw/refs/heads/main/sub/normal/10ium-V2Hub3-vless",
  "https://github.com/10Dream/sub-mod/raw/refs/heads/main/sub/normal/10ium-V2Hub3-trojan",
  "https://github.com/10Dream/sub-mod/raw/refs/heads/main/sub/normal/10Dream-VpnClashFaCollector-mixed.txt",
  "https://github.com/MustafaBaqer/VestraNet-Nodes/raw/refs/heads/main/subscriptions/mix-normal.txt",
  "https://github.com/Firmfox/Proxify/raw/refs/heads/main/v2ray_configs/separated_by_protocol/trojan.txt",
  "https://github.com/Firmfox/Proxify/raw/refs/heads/main/v2ray_configs/separated_by_protocol/vless.txt",
  "https://github.com/Firmfox/Proxify/raw/refs/heads/main/v2ray_configs/separated_by_protocol/vmess.txt",
  "https://github.com/miladtahanian/V2RayCFGDumper/raw/refs/heads/main/sub.txt",
  "https://github.com/ShatakVPN/ConfigForge-V2Ray/raw/refs/heads/main/configs/all.txt",
  "https://github.com/0xRadikal/Free-v2ray-Configs/raw/refs/heads/main/all/configs.txt",
  "https://github.com/roosterkid/openproxylist/raw/refs/heads/main/V2RAY_RAW.txt",
  "https://github.com/miladtahanian/V2RayScrapeByCountry/raw/refs/heads/main/output_configs/Trojan.txt",
  "https://github.com/miladtahanian/V2RayScrapeByCountry/raw/refs/heads/main/output_configs/Vless.txt",
  "https://github.com/miladtahanian/V2RayScrapeByCountry/raw/refs/heads/main/output_configs/Vmess.txt",
  "https://github.com/MhdiTaheri/V2rayCollector/raw/refs/heads/main/sub/mix",
  "https://github.com/barry-far/V2ray-Config/raw/refs/heads/main/Splitted-By-Protocol/trojan.txt",
  "https://github.com/barry-far/V2ray-Config/raw/refs/heads/main/Splitted-By-Protocol/vless.txt",
  "https://github.com/barry-far/V2ray-Config/raw/refs/heads/main/Splitted-By-Protocol/vmess.txt",
  "https://github.com/iboxz/free-v2ray-collector/raw/refs/heads/main/main/mix.txt",
  "https://github.com/iboxz/free-v2ray-collector/raw/refs/heads/main/main/mix"
];

// ============================================================
// الـ IP الخاص بك (التوزيع بالتناوب)
// ============================================================
const HOST_POOL = [
  "104.18.27.137",
];

let hostIndex = 0;

const FETCH_TIMEOUT_MS = 30_000;
const CONCURRENCY_LIMIT = 30;

function timeoutSignal(ms) {
  if (typeof AbortSignal !== 'undefined' && typeof AbortSignal.timeout === 'function') {
    return AbortSignal.timeout(ms);
  }
  const controller = new AbortController();
  setTimeout(() => controller.abort(), ms);
  return controller.signal;
}

function isPrintableText(text) {
  if (!text) return false;
  const cleaned = text.replace(/[\x00-\x08\x0B\x0C\x0E-\x1F\x7F]/g, '');
  return cleaned.length / text.length > 0.9;
}

function decodeIfBase64(text) {
  try {
    const cleaned = text.trim().replace(/\s+/g, '');
    const normalized = cleaned.replace(/-/g, '+').replace(/_/g, '/');
    const padded = normalized.padEnd(Math.ceil(normalized.length / 4) * 4, '=');
    const decoded = Buffer.from(padded, 'base64').toString('utf8');

    if (!isPrintableText(decoded)) return text;

    if (
      decoded.includes('vless://') ||
      decoded.includes('trojan://') ||
      decoded.includes('vmess://')
    ) {
      return decoded;
    }
  } catch (_) {}
  return text;
}

function b64DecodeUtf8(b64) {
  try {
    b64 = b64.trim().replace(/\s/g, '');
    b64 += '='.repeat((4 - (b64.length % 4)) % 4);
    return Buffer.from(b64, 'base64').toString('utf8');
  } catch { return null; }
}

function b64EncodeUtf8(str) {
  return Buffer.from(str, 'utf8').toString('base64');
}

async function fetchWithLimit(urls, limit) {
  const results = [];
  for (let i = 0; i < urls.length; i += limit) {
    const batch = urls.slice(i, i + limit);
    const batchResults = await Promise.allSettled(
      batch.map((url) =>
        fetch(url, {
          headers: { 'user-agent': 'Mozilla/5.0' },
          signal: timeoutSignal(FETCH_TIMEOUT_MS),
        })
          .then((res) => {
            if (!res.ok) {
              console.warn(`[FAIL ${res.status}] ${url}`);
              return '';
            }
            return res.text();
          })
          .catch((err) => {
            console.warn(`[ERR] ${url} — ${err?.message ?? String(err)}`);
            return '';
          })
      )
    );
    results.push(...batchResults);
  }
  return results;
}

async function main() {
  console.log("Starting fetch...");
  const results = await fetchWithLimit(sourceUrls, CONCURRENCY_LIMIT);

  const texts = results
    .filter((r) => r.status === 'fulfilled' && r.value)
    .map((r) => r.value);

  if (texts.length === 0) {
    console.error('⚠️ All sources failed to fetch.');
  }

  const filteredConfigs = [];
  
  // Sets ديال الفلترة (Dedupe) - كنستعملو مفاتيح مركبة باش نكونو دقيقين
  const seenVless = new Set();
  const seenVmess = new Set();
  const seenTrojan = new Set();

  for (const text of texts) {
    let decodedText = text;

    if (!text.includes('vless://') && !text.includes('trojan://') && !text.includes('vmess://')) {
      decodedText = decodeIfBase64(text);
    }

    const lines = decodedText.split(/\r?\n/);

    for (let line of lines) {
      line = line.trim();
      if (!line) continue;

      // ==========================================
      // 1. VMess
      // ==========================================
      if (line.startsWith('vmess://')) {
        try {
          const cleanLine = line.split('#')[0];
          const b64 = cleanLine.slice(8).trim();
          const jsonStr = b64DecodeUtf8(b64);
          if (!jsonStr) continue;

          const config = JSON.parse(jsonStr);

          // فلترة: WS + TLS + Port 443
          if (
            String(config.net || '').toLowerCase() !== 'ws' ||
            String(config.tls || '').toLowerCase() !== 'tls' ||
            parseInt(config.port, 10) !== 443
          ) {
            continue;
          }

          const server = String(config.add || '').toLowerCase().trim();
          if (!server) continue;

          // Dedupe: ID + Path + Host
          const vmessKey = `${config.id}_${config.path}_${config.host}`.toLowerCase();
          if (seenVmess.has(vmessKey)) continue;
          seenVmess.add(vmessKey);

          // توزيع الـ Host (IP)
          const currentHost = HOST_POOL[hostIndex % HOST_POOL.length];
          hostIndex++;

          // ✅ الحفاظ على SNI و Host الأصليين قبل تغيير الدومين
          if (!config.host) config.host = server;
          if (!config.sni) config.sni = server;
          
          config.add = currentHost;

          filteredConfigs.push('vmess://' + b64EncodeUtf8(JSON.stringify(config)));
        } catch (_) {}
        continue;
      }

      // ==========================================
      // 2. VLESS / Trojan
      // ==========================================
      const cleanLine = line.split('#')[0].trim();
      if (!cleanLine.startsWith('vless://') && !cleanLine.startsWith('trojan://')) {
        continue;
      }

      try {
        const url = new URL(cleanLine);
        const protocol = url.protocol.replace(':', '').toLowerCase();
        const port = url.port === '' ? 443 : parseInt(url.port, 10);

        // فلترة: WS + TLS + Port 443
        if (
          (protocol !== 'vless' && protocol !== 'trojan') ||
          url.searchParams.get('type') !== 'ws' ||
          url.searchParams.get('security') !== 'tls' ||
          port !== 443
        ) {
          continue;
        }

        const username = url.username.toLowerCase().trim();
        if (!username) continue;

        const originalHostname = url.hostname.toLowerCase();

        // Dedupe: Username + Path + SNI
        const path = url.searchParams.get('path') || '/';
        const sni = url.searchParams.get('sni') || originalHostname;
        const dedupeKey = `${username}_${path}_${sni}`;

        if (protocol === 'vless') {
          if (seenVless.has(dedupeKey)) continue;
          seenVless.add(dedupeKey);
        }
        if (protocol === 'trojan') {
          if (seenTrojan.has(dedupeKey)) continue;
          seenTrojan.add(dedupeKey);
        }

        // توزيع الـ Host (IP)
        const currentHost = HOST_POOL[hostIndex % HOST_POOL.length];
        hostIndex++;

        // ✅ الحفاظ على SNI و Host الأصليين قبل تغيير الدومين
        if (!url.searchParams.has('sni')) url.searchParams.set('sni', originalHostname);
        if (!url.searchParams.has('host')) url.searchParams.set('host', originalHostname);

        url.hostname = currentHost;

        let finalConfig = url.toString();
        const originalRemark = line.split('#')[1];
        if (originalRemark) finalConfig += `#${originalRemark}`;

        filteredConfigs.push(finalConfig);
      } catch (_) {}
    }
  }

  if (filteredConfigs.length === 0) {
    console.error('⚠️ No configs found! All sources failed or filter too strict.');
    try {
      const old = await fs.readFile('sub.txt', 'utf8');
      if (old.trim().length > 0) {
        console.warn('ℹ️ Keeping existing sub.txt to avoid wiping the subscription.');
        return;
      }
    } catch (_) {}
    process.exit(1);
  }

  await fs.writeFile('sub.txt', filteredConfigs.join('\n'), 'utf8');
  console.log(`✅ Saved ${filteredConfigs.length} configs to sub.txt`);
}

main().catch((err) => {
  console.error(err);
  process.exit(1);
});
