## VULN-01 Content Security Policy (CSP) Header Not Set

- Site / URL: http://localhost:3000
- Severidade (ZAP): Médio. :contentReference[oaicite:7]{index=7}
- Confidence (ZAP): Alto / Médio (ver relatorio).
- Evidence:
  - Response headers from GET / : no CSP header present. (ver seção `Content Security Policy (CSP) Header Not Set`). :contentReference[oaicite:8]{index=8}
- Mapeamento OWASP: A02 — Security Misconfiguration.
- Reproduzível:
  1. curl -I http://localhost:3000
  2. Verificar ausência do header `Content-Security-Policy`.
- Recomendações iniciais:
  - Definir CSP adequada (p.ex. `default-src 'self'; script-src 'self' 'unsafe-inline'` apenas se necessário — preferir não usar 'unsafe-inline').
  - Implementar header via `next.config.js` (exemplo abaixo).
- Status: Open

## VULN-02 CORS / Configuração Incorreta Entre Domínios

- Site / URL: http://localhost:4000 e http://localhost:3000 (relatório mostra ambos). :contentReference[oaicite:9]{index=9}
- Severidade: Médio.
- Evidence:
  - Header `Access-Control-Allow-Origin: *` em algumas respostas. :contentReference[oaicite:10]{index=10}
- Mapeamento OWASP: A02 — Security Misconfiguration.
- Reproduzível:
  1. curl -I http://localhost:4000/somepath
  2. Verificar `Access-Control-Allow-Origin: *`.
- Recomendações iniciais:
  - Restringir origem para domínios confiáveis.
  - Aplicar políticas CORS no lado servidor (ou middleware) apenas para rotas API que precisam.
- Status: Open

## VULN-03 Missing Anti-clickjacking Header (X-Frame-Options or CSP frame-ancestors)

- Site / URL: http://localhost:3000
- Severidade: Médio. :contentReference[oaicite:11]{index=11}
- Evidence: ausência de `X-Frame-Options` / `frame-ancestors` na resposta. :contentReference[oaicite:12]{index=12}
- Mapeamento OWASP: A02 / A01 dependendo do impacto (principalmente Security Misconfiguration).
- Reproduzível:
  - curl -I http://localhost:3000 | grep -i frame
- Recomendações iniciais:
  - Adicionar `X-Frame-Options: DENY` ou CSP `frame-ancestors 'none'`.
- Status: Open

## VULN-04 Directory browsing (Navegação no Diretório)

- Site / URL: http://localhost:3000/\_next/static/chunks/app/md-render.html/page.js/
- Severidade: Médio/Baixo (conforme relatório). :contentReference[oaicite:13]{index=13}
- Evidence: diretório retornando listagem/response indicando recursos acessíveis. :contentReference[oaicite:14]{index=14}
- Mapeamento OWASP: A01 — Broken Access Control (exposição de arquivos) / A02.
- Reproduzível:
  1. Acessar a URL acima no navegador ou curl.
- Recomendações iniciais:
  - Garantir que o servidor não permita `Options Indexes` (ou equivalente).
  - Ajustar servidor / build para prevenir exposição de diretórios (servir apenas arquivos estáticos conhecidos).
- Status: Open

## VULN-05 X-Powered-By header exposure

- Site / URL: http://localhost:3000
- Severidade: Baixo (informativo) — porém facilita fingerprinting. :contentReference[oaicite:15]{index=15}
- Evidence: header `X-Powered-By: Next.js` presente. :contentReference[oaicite:16]{index=16}
- Mapeamento OWASP: A02 — Security Misconfiguration (information leakage).
- Reproduzível:
  - curl -I http://localhost:3000 | grep -i x-powered-by
- Recomendações iniciais:
  - Remover ou suprimir header em ambiente de produção.
  - Configurar host/serviço para não expor stack.
- Status: Open
