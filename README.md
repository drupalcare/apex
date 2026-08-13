# APEX

APEX is the platform **kernel** that every APEX sub-module and the APEX theme
build on. It ships shared contracts and platform primitives — not end-user
features: sub-module discovery, theme detection, region metadata and
validation, CSS assembly and security utilities, layout math, and cache
management.

Composer installs it automatically with any APEX sub-module. It is not useful
standalone, and it deliberately contains no feature logic of its own — that
lives in the sub-modules.

## Requirements

- Drupal 11.3+ (forward-compatible with Drupal 12)
- PHP 8.3+

## What it provides

Every service exposes an interface alias for autowiring. Inject the specific
interface you need.

### Feature discovery

- **`apex.feature_registry`** (`ApexFeatureRegistryInterface`) — collects every
  service tagged `apex.feature_provider`, so the parent never hardcodes a list
  of sub-modules. Disabled modules drop out of the container automatically.
- **`apex.feature_form_order_resolver`**
  (`FeatureFormOrderResolverInterface`) — orders theme-settings sections by
  dependency and weight, keeping presentation out of the registry.

### Theme and regions

- **`apex.theme_detection`** (`ThemeDetectionServiceInterface`) — whether the
  active theme is APEX, directly or through sub-theme inheritance. Cached per
  active theme.
- **`apex.regions_provider`** (`RegionsProviderInterface`) — canonical region
  IDs, labels, CSS selectors, and UI groups. The single source of truth for
  region metadata.
- **`apex.region_validator`** (`RegionValidatorServiceInterface`) — whether a
  region holds genuinely renderable content, so empty render arrays do not
  produce empty wrapper markup.
- **`apex.region_block_status`** (`RegionBlockStatusServiceInterface`) — which
  regions hold placed blocks for the active theme.

### CSS utilities

- **`apex.css.inline_injector`** (`ApexInlineCssInjectorInterface`) — emits
  inline `<style>` with the APEX cascade-layer ladder prepended, honouring the
  parse-order contract. Always route inline `@layer apex.*` CSS through this.
- **`apex.css.file_manager`** (`ApexCssFileManagerInterface`) — writes and
  serves generated CSS files.
- **`apex.css.region_variable_renderer`**
  (`RegionVariableCssRendererInterface`) — shared region-scoped custom-property
  renderer, used by `apex_schemes` and `apex_color` to emit identical
  selectors, format, and layer wrapping.

### Security

- **`apex.security.svg_sanitizer`** (`SvgSanitizerInterface`) — allowlist SVG
  sanitizer for user-supplied icons.
- **`apex.security.css_class_validator`** (`CssClassValidatorInterface`) —
  validates and sanitizes CSS class strings before they reach markup.
- **`apex.security.css_dimension_sanitizer`**
  (`CssDimensionSanitizerInterface`) — allowlists CSS units and clamps numeric
  ranges for any value that becomes dynamic CSS. Config can be imported or
  hand-edited outside the settings form, so this is the shared guard against
  CSS injection through stored dimensions.

### Layout math

- **`apex.fluid_scale_calculator`** (`FluidScaleCalculatorInterface`) —
  Utopia-style `clamp()` generation for fluid type and space.
- **`apex.site_breakpoint`** (`SiteBreakpointServiceInterface`) — the
  user-selected breakpoint for responsive CSS.

### Cache

- **`apex.cache_manager`** (`CacheManagerInterface`) — targeted cache
  invalidation helpers. Prefer cache tags; the broad flush methods are intended
  for explicit admin operations only.

### Twig

- **`apex.twig_extension`** — exposes region-content checks to templates.

## Extending

Sub-modules describe themselves to the platform rather than the parent knowing
them:

- Implement `ApexFeatureProviderInterface` (or extend `ApexFeatureProviderBase`)
  and tag the service `apex.feature_provider`.
- Read settings through `ApexSettingsReaderBase`.
- Build theme-settings UI through `ApexThemeSettingsSubscriberBase`.

These three base classes are marked `@api` — treat them as stable extension
points.

## Hooks

Hooks are object-oriented (Drupal 11.1+), auto-discovered from `src/Hook/`.
There is no procedural hook registration.

## Testing

```bash
# Unit tests
vendor/bin/phpunit -c core/phpunit.xml.dist modules/apex/tests/src/Unit

# Static analysis and coding standards
vendor/bin/phpstan analyse -c modules/apex/phpstan.neon modules/apex/src
vendor/bin/phpcs --standard=Drupal,DrupalPractice modules/apex/src
```

## License

GPL-2.0-or-later
