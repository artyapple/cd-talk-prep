# Container Days Talk: slides description and script

---

## Slide 0: Cover 
### What is on the slide:
**Title:** API-First: Because Life’s Too Short for Boilerplate

Artur Iablokov & Benedikt Dahm  
Lufthansa Industry Solutions

---

## Slide 1: Intro
### What is on the slide:  
**Title:** Intro 

**Bullets:**
- We build microservices in **Go** with **API-first**
- **Myth →** “API-first slows you down”
- **Reality →** Removes **rework** & **boilerplate**

---

## Slide 2: API-First for Developers
### What is on the slide:  
**Title:** API-First for Developers: What It *Really* Means  
**Bullets:**  
- Write **contracts first**, not boilerplate handlers  
- Generate safe **code** (types, routes, clients)  
- Compiler, not meetings, enforces the contract  
- Less glue code, more feature code

---

## Slide 3: Codegen Tools Map
### What is on the slide:  
**Title:** Codegen Tools in Go (OpenAPI as Example)  
**Bullets:**  
- **oapi-codegen** — pragmatic, Go-idiomatic, overlays, strict server.  
- **ogen** — prescriptive, performance-focused, strong validation.  
- **OpenAPI Generator** — multi-language support, SDKs for clients.  
- **Goa** — design-first, full-service framework.

---

## Slide 4 — Strict vs Non-Strict Server (Typed vs Classic)
### What is on the slide:
**Title:** Strict vs Non-Strict Server (Typed vs Classic)

**Quick compare (3 bullets each):**

**Strict Server**
- Typed ops: `RequestObject → ResponseObject` 
```
func (s *Server) FindDevices(ctx context.Context, req FindDevicesRequestObject) (FindDevicesResponseObject, error)
```
- Compile-time contract enforcement
- Clear status/media types as Go types

**Non-Strict (classic)**
- Familiar `http.ResponseWriter, *http.Request`
```
func (s *Server) FindDevices(w http.ResponseWriter, r *http.Request)
```
- Max flexibility (middleware, custom routing)
- Easier drop-in for legacy handlers

---

## Slide 5: Extensions That Speak Go
### What is on the slide: 
**Title:** Extensions that speak Go  

**Bullets (keep tiny):**
- Dozens of built-in `x-*` extensions
- Map schemas to **real Go code**
- Result: **fewer DTOs**, spec stays the source of truth

```yaml
// openapi.yaml
Device:
  type: object
  properties:
  id:
    type: string
    x-oapi-codegen-extra-tags:
      bson: "_id,omitempty"
    x-go-type: primitive.ObjectID
    x-go-type-import:
      path: go.mongodb.org/mongo-driver/bson/primitive
  name:
    type: string   
  required: [ id, name ]
```
======>
```
// api.gen.go
type Device struct {
	Id    primitive.ObjectID  `bson:"_id,omitempty" json:"id"`
	Name  string              `json:"name"`
}
```

---

## Slide 6: One Spec, Many Views (Overlays) 
### What is on the slide:   
**Title:** One Spec, Many Views (Overlays)

**Bullets:**
- **New OpenAPI Overlay** spec v1.0.0 - published 10/2024 
- Patch or slice a spec *without forking*  
- `oapi-codegen` already supports it  

```
overlay: 1.0.0
info:
  title: "Client-slice overlay"
  version: "1.0.0"

actions:
  # 1. Tag only the endpoint we care about
  - target: "$.paths['/reports'].get.tags"
    update:
      - client-slice

  # 2. (Optional) remove everything else
  - target: "$.paths['/admin']"
    remove: true
  - target: "$.paths['/users']"
    remove: true
```

---

## Slide 7: Multiple packages vs one package 
### What is on the slide:
**Title:** Multiple packages vs one package
**Bullets (tiny, 3 lines):**
- Split specs → duplicate models (`device.Device`, `report.Device`…)  
- **Same-package import mapping** = one canonical `Device`  
- Choice: multiple packages vs one package

```
package: api
output: server.gen.go
generate:
  models: true
  chi-server: true
output-options:
  skip-prune: true
import-mapping:
  device.yaml: "-"
```  
```
package: api
output: device.gen.go
generate:
  models: true
output-options:
  skip-prune: true
```

---

## Slide 8 — Wrap up: Principles > Tools
### What is on the slide:
**Title:** Wrap up: Principles > Tools
**Bullets (keep tight):**
- What mattered in my projects: type-safe handlers, idiomatic Go models, one contract → many generated views
- **Your context may differ** — your constraints drive tools
- The **patterns travel** across ecosystems:
  - OpenAPI → oapi-codegen / ogen / OpenAPI Generator
  - AsyncAPI → asyncapi-codegen
  - Protobuf/gRPC → Buf + ConnectRPC
  - GraphQL → gqlgen
- Constant: **API-first** → design contract, **generate & validate**, **remove rework and boilerplate**.