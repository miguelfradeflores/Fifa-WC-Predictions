# **🧾 PRD — World Cup 2026 Prediction Game (Coderoad)**

---

## **1\. Investment Themes**

* Employee Engagement & Culture  
* Lightweight Internal Products  
* Gamification & Participation  
* Data-driven Interaction (real-time updates)

---

## **2\. Investment & Core Value Proposition**

Construir una **plataforma interna ultra simple** que permita a los empleados de Coderoad participar en una **polla del mundial 2026**, generando engagement a través de predicciones, competencia y ranking en tiempo real.

El valor principal es:

* Participación masiva  
* Experiencia simple y divertida  
* Feedback inmediato (ranking dinámico)

---

## **3\. Defining the Problem (Buyer Statement)**

Como empresa 100% remota, Coderoad necesita generar instancias de interacción social y engagement entre empleados, especialmente en eventos globales como el mundial.

Actualmente:

* No existe un sistema centralizado  
* Las apuestas son manuales o dispersas  
* No hay transparencia ni automatización

---

## **4\. Client (Check Writer)**

* Internal Leadership (People / Culture / Ops)  
* Stakeholders: Engineering \+ Employees

---

## **5\. Specific Business Problem**

* Falta de herramientas internas para engagement  
* Procesos manuales de apuestas  
* Falta de transparencia en resultados  
* No existe sistema automático de scoring

---

## **6\. Success Measurement**

* ≥ 70% de empleados registrados  
* ≥ 60% de participación en predicciones  
* ≥ 90% de partidos con predicciones registradas  
* Tiempo de actualización de ranking \< 60 min post-partido  
* 0 errores críticos en cálculo de puntaje

---

## **7\. Value Proposition**

Para usuarios:

* Predicciones simples  
* Ranking en tiempo real  
* Competencia divertida

Para negocio:

* Engagement interno  
* Cultura organizacional  
* Costo casi cero

---

## **8\. Scenarios**

### **✅ Happy Path**

1. Usuario inicia sesión con Google  
2. Ve partidos disponibles  
3. Realiza predicciones (todo el torneo)  
4. Sistema bloquea edición 15 min antes  
5. API entrega resultado real  
6. Sistema calcula puntos automáticamente  
7. Ranking se actualiza

---

### **🔁 Edición de predicción**

* Usuario puede modificar predicción ilimitadamente  
* Hasta 15 min antes del inicio

---

### **❌ No participación**

* Usuario no predice → no suma puntos  
* UI muestra estado: “No participó”

---

### **⚠️ API falla**

* Resultado no disponible  
* Fallback: actualización manual (backend)

---

### **🔓 Post cierre**

* Predicciones se vuelven visibles para todos

---

## **9\. Go-To-Market Strategy (GTM)**

* Lanzamiento interno (Coderoad)  
* Comunicación vía Slack / Email  
* Incentivo: badges \+ reconocimiento

---

## **10\. What Success Looks Like**

* Usuarios revisan dashboard diariamente  
* Ranking genera competencia activa  
* Conversaciones internas sobre resultados  
* Alta participación en fases finales

---

## **11\. Definition of Done**

* Login con Google funcional  
* Lista completa de partidos disponible  
* Predicciones guardadas por usuario  
* Bloqueo automático pre-partido (15 min)  
* Integración con API de resultados  
* Cálculo automático de puntaje  
* Ranking global y por fase visible  
* Predicciones visibles post cierre  
* Badges calculados y asignados

---

## **12\. Constraints**

* MVP en 3 horas → simplicidad extrema  
* Reglas hardcoded (TBD valores)  
* Sin panel admin  
* Dependencia de API externa de resultados  
* Desktop-first  
* Sin notificaciones  
* Sin pagos

---

## **13\. High-Level Features**

---

### **🧠 Rule Engine (Simplificado)**

* Reglas hardcoded (TBD valores)  
* Evaluación por partido:  
  * Marcador exacto  
  * Ganador acertado  
  * Gol acertado (máx 1\)  
  * Predicción única  
* Reglas acumulativas (todo suma)  
* Bonos por fase (todo o nada)

---

### **🔗 Data Aggregation Layer**

* Consumo de API externa de partidos  
* Datos:  
  * Equipos  
  * Fecha/hora  
  * Resultado final  
* Trigger de actualización:  
  * Post partido  
  * Manual fallback

---

### **📊 Scoring Engine**

* Cálculo por usuario por partido  
* Agregación total:  
  * Ranking global  
  * Ranking por fase  
* Cálculo incremental (por partido finalizado)

---

### **🧑‍💻 User Experience**

**Dashboard incluye:**

1. Próximos partidos (CTA a apostar)  
2. Ranking global  
3. Ranking por fase  
4. Tus predicciones  
5. Historial

---

### **📝 Predictions System**

* Usuario puede:  
  * Apostar todo el torneo  
  * Editar predicciones  
* Restricción:  
  * Bloqueo 15 min antes

---

### **🔒 Visibility Rules**

* Antes del cierre:  
  * Predicciones ocultas  
* Después:  
  * Predicciones visibles para todos

---

### **🏆 Badges System**

* 🥇 World Champion (Top 1\)  
* 🥈 Elite Predictor (Top 2\)  
* 🥉 Sharp Eye (Top 3\)  
* 🧠 Más marcadores exactos  
* 🎯 Más goles acertados

---

### **🔐 Authentication**

* Google Login only

---

## **14\. Risks & Mitigation**

| Riesgo | Impacto | Mitigación |
| ----- | ----- | ----- |
| API externa falla | Alto | Fallback manual |
| Lógica de puntaje mal definida | Alto | Validación con ejemplos |
| Scope creep | Alto | Hardcode \+ sin admin |
| Performance ranking | Medio | Cálculo incremental |
| Ambigüedad en reglas | Alto | Documentar claramente |

---

## **15\. Dependencies**

### **Sistemas**

* API de resultados de fútbol (TBD)  
* Google Auth

### **Equipos**

* Engineering (full ownership)  
* Producto (definición reglas)

---

## **16\. Open Questions / TBD**

* Valores exactos de puntaje (actualmente TBD)  
* API oficial a utilizar  
* Timezone handling final  
* Validación final de reglas con stakeholders

---

## **17\. Decisiones Clave de MVP**

* ✅ Reglas hardcoded  
* ✅ Sin admin panel  
* ✅ Sin notificaciones  
* ✅ Sin pagos  
* ✅ Sin multi-grupo  
* ✅ Single global group  
* ✅ Desktop-first

---

## **18\. Arquitectura Simplificada (MVP-ready)**

* Frontend: Web app (React/Next)  
* Backend: API simple (Node/Firebase/Supabase)  
* DB:  
  * Users  
  * Matches  
  * Predictions  
  * Scores  
* Scheduler:  
  * Update results  
  * Recalculate ranking

---

## **19\. Lo que este MVP habilita (Phase 2\)**

* Reglas configurables  
* Multi-grupo  
* Notificaciones  
* App mobile  
* Integración social  
* Betting system (opcional)

