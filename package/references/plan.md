# Sonar Check — Plano de Remediação
**Projeto:** {{projectKey}}
**Modo de análise:** {{analysisMode}}
**Gerado em:** {{timestamp}}
**Arquivos analisados:** {{deltaFiles}}
**Quality Gate:** FALHOU
**Violações encontradas:** {{totalFindings}}
{{degradationWarning}}

---

## 📋 Tarefas de Resolução

{{tasks}}

---

## 📊 Diagnóstico Completo

| Severidade | Arquivo | Linha | Mensagem |
|-----------|---------|-------|----------|
{{findingsTable}}

---

## 🔧 Remediação Detalhada

{{remediationSections}}

---

## 📈 Cobertura

- **Cobertura atual (New Code):** {{newCoverage}}% (threshold: {{gateThreshold}}%)
- **Gap:** {{coverageGap}}pp
- **Arquivos sem cobertura no delta:** {{uncoveredFiles}}

---

## 🔒 Hotspots de Segurança (informativo — não bloqueiam o gate)

Revisar em: {{sonarCloudUrl}}/security_hotspots?id={{projectKey}}

| Arquivo | Linha | Mensagem |
|---------|-------|----------|
{{hotspotsTable}}
