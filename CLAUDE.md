# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What is this workspace?

This is the **DRUCS** (Dropsa Remote Universal Controller System) monorepo workspace — a collection of independent Git repositories for DropsA's industrial IoT lubrication platform. Each subfolder is a separate repo.

---

## Repository Map

| Folder | Status | Tech | Role |
|--------|--------|------|------|
| `import-web-app` | **Active** | PHP 8.4, Laravel 12 | **Core API** — used by both web and mobile |
| `frontend-web` | **Retired** | Angular 17 | Old web dashboard (replaced by drucs-v2) |
| `mobile` | **Active** | Flutter/Dart | iOS + Android app |
| `drucs-gateway-new` | **Stalled** | Java 17, Spring Boot 3 | Device TCP gateway |
| `drucs-web-services` | **Deprecated** | PHP (flat files) | Old mobile API — no longer in use |
| `S-00365-firmware` | **Active** | C, FreeRTOS, LVGL | STM32L496 device firmware |
| `MEV2_bootloader` | **Active** | C, STM32L4 HAL | STM32L496 OTA bootloader |
| `S-00349-LP_sensor` | **Active** | C, STM32F1 HAL | Level/pressure sensor firmware |
| `Omega` | Active | C, STM32CubeMX | STM32 project (OmegaPump) |
| `drucs-iac` | Active | Terraform + Terragrunt | AWS infrastructure (ECS, RDS, S3, SQS…) |
| `drucs-docs` | Active | MkDocs | Platform documentation site |
| `github-actions` | Active | YAML | Shared CI/CD workflow templates |
| `fastlane-ios-certs` | Active | Fastlane Match | iOS code signing certificates |
| `architecture` | Reference | Markdown | Architecture overview |
| `P30015760-Maxtreme` | Placeholder | — | Empty/early-stage |

---

## System Architecture

```
STM32 Device (S-00365-firmware)
  │  WiFi via ESP32 co-processor
  │  TCP + AES-128-CFB (per-device key, zero IV — known weakness)
  ▼
DRUCS Gateway (drucs-gateway-new)  ←→  AWS SQS
  │  Internal AWS network (ECS service mesh)
  ▼
Core API (import-web-app) ←→ PostgreSQL (RDS) + Redis (ElastiCache) + S3
  │  HTTPS + JWT
  ├──▶ Frontend v2 (drucs-v2 — React 19 + Vite, at app.dropsa.app/v2/)
  └──▶ Mobile App (mobile — Flutter, iOS + Android)

BLE path: STM32 ←→ Mobile app directly (PIN-protected, 4–6 digits)
```

**Key architectural rules:**
- `import-web-app` is the single backend for both web and mobile. `drucs-web-services` is deprecated and must not be extended.
- **Web leads, mobile follows** — new features are designed in the React web dashboard first, then adapted to Flutter.
- Commands from the cloud reach the device only on the next heartbeat response (poll-based, not push).

---

## Device Variable ID System

Every device communicates using numeric variable IDs. This is the shared contract between firmware, gateway, API, and UI:

| Range | Type |
|-------|------|
| 1 – 99,999 | Settings (user-configurable) |
| 100,000 – 199,999 | Live data (read-only telemetry) |
| 200,000 – 300,000 | Commands (actions) |

Key IDs: `100100` = total motor hours (seconds), `100101` = pump rotations, `100017` = reservoir level %, `100015`/`100016` = active warnings/alarms (semicolon-separated event IDs), `200001` = start lubrication, `200002` = stop, `200010` = OTA firmware update.

Full map: `drucs-docs/docs/platform/variable-map.md`

## Product Variants

The same firmware (`S-00365-firmware`) is compiled with different `#define` flags per product:

| Code | Product | BLE Prefix |
|------|---------|------------|
| BC | BravoCompact 4.0 | `BC_` |
| BR | Bravo 4.0 | `BR_` |
| SM | Smart 4.0 | `SM_` |
| OM | OmegaPump | `OM_` |
| ST | SilentTrack | `ST_` |
| NV | VipAir 4.0 | `NV_` |
| MA | Maxtreme | `MA_` |
| VP | VIP6 | `VP_` |

---

## Commands by Repository

### `import-web-app` — Core API (Laravel 12, PHP 8.4)

```bash
# Start
cp .env.example .env
docker compose --env-file .env build
docker compose --env-file .env up -d

# Tests — prepare DB first, then run
docker exec -it dropsa-web-app php artisan migrate:fresh --env=test --seed
docker exec -it dropsa-web-app php artisan test --env=test
docker exec -it dropsa-web-app php artisan test --env=testing --filter=UserControllerTest
docker exec -it dropsa-web-app php artisan test --env=testing --filter=it_can_invite_user

# Code style
docker exec -it dropsa-web-app ./vendor/bin/pint

# Swagger docs
docker exec -it dropsa-web-app php artisan l5-swagger:generate

# Queue workers (queues: general, emails, event-report, summary-report, device-events)
docker exec -it dropsa-web-app php artisan queue:work --queue={QUEUE_NAME}

# Translations (managed via Google Sheets)
docker exec -it dropsa-web-app php artisan translations:parse
docker exec -it dropsa-web-app php artisan translations:update
```

Local services: API at `http://localhost`, email inspector (Mailpit) at `http://localhost:8025`. AWS services (S3, SQS, SNS, ElastiCache) are mocked via **Localstack** in local dev.

Architecture: **Controller → Service → Repository**. Business logic lives in `app/Dropsa/`. Repositories are read-only; writes go through Laravel models. DTOs use `readonly` properties.

### `mobile` — Flutter App

```bash
# Code generation (after model/route/asset changes)
make gen        # dart run build_runner build --delete-conflicting-outputs
make loc        # regenerate l10n (app_localizations.dart)

# Dev build (APK)
./build_dev.sh

# Prod build (AAB)
./build_prod.sh
```

Architecture: **BLoC pattern** — prefer Cubit over Bloc. Structure: `lib/feature/<feature_name>/{cubit,data,screen,widget,domain}`. All generated files are committed. Routing via `auto_route`. Environments: dev / stg / prod — base URLs in `lib/core/api/environment/environment_data.dart`. iOS signing uses `fastlane-ios-certs` (match, `com.dropsa.drucs`).

### `S-00365-firmware` — STM32 Device Firmware

Builds via **STM32CubeIDE headless** in CI. Build configurations: `Debug`, `Release`, `DUT_CLI`, `TB_CLI`.

```bash
# CI runs (requires STM32CubeIDE Docker image registry.promwad.net/stm32/stm32cubeide:1.19.0)
headless-build.sh -data /tmp/stm-workspace -build S-00365-ModularElectronicsV2/Release

# Unit tests (run locally with CMake + Unity)
cd Drivers/ESP32/tests && cmake -S . -B build && cmake --build build && ./build/esp32_test
cd tests && cmake -S . -B build && cmake --build build && ctest --test-dir build --verbose
```

Structure: `BSP/` (Board Support Package — BLE, WiFi, LCD, sensors, OTA), `Core/` (application logic, FreeRTOS tasks), `Drivers/` (HAL, ESP32, external ICs).

### `drucs-gateway-new` — Device Gateway (Spring Boot 3, Java 17)

```bash
./gradlew bootJar          # builds drucs-gateway.jar
./gradlew test             # runs tests (Testcontainers + MariaDB)
./gradlew bootRun          # local run
```

Listens on TCP port `1256` at `s2.dropsa.com`. Decrypts AES-128-CFB device messages, parses variable IDs, stores to DB, forwards commands. Uses AWS SNS for push notifications.

### `drucs-iac` — Infrastructure (Terraform 1.7.3 + Terragrunt 0.55.2)

```bash
# In a region folder (e.g., terragrunt/dev/aws/eu-west-2/)
terragrunt graph-dependencies   # visualise module dependency order
terragrunt apply                # apply a single module
terragrunt run-all apply        # apply all modules in dependency order

# Clean cache
find . -type d -name ".terragrunt-cache" -prune -exec rm -rf {} \;
```

Module apply order for a new env: (1) vpc, ecr, certificate-manager, sqs → (2) cdn, rds, elasticache → (3) s3, elb, ecs, ssm-parameter-store, jump-host, ci-cd.

SSM secrets to add manually after provisioning: `mail/password`, `drucs-core-api/app-key`, `drucs-core-api/jwt-secret`, `big-data-cloud-api-key`, `open-weather-api-key`, `google-recaptcha-key/secret`.

### `drucs-docs` — MkDocs Documentation Site

```bash
pip install mkdocs-material
mkdocs serve      # local preview
mkdocs build      # build static site to site/
```

Deployed at `enghelper.dropsa.net/docs/`.
