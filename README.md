# Kustomize Edit

Updates kustomize overlays with new image tags, labels, and annotations.

## Features

- 🏷️ **Image updates** - Set new image tags
- 📝 **Label management** - Add/update labels
- 🔖 **Annotations** - Set deployment metadata
- 📅 **Automatic timestamps** - Track deployment times
- 🎯 **Smart detection** - Handles various kustomize patterns
- 🔧 **ConfigMap injection** - Inject runtime values into ConfigMaps

## Usage

```yaml
- name: Update kustomize overlay
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/production
    image: backend
    tag: v1.2.3
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `overlay_dir` | Path to kustomize overlay | ✅ | - |
| `debug` | Enable debug output | ❌ | `false` |
| `image` | Image name (without tag) | ❌ | - |
| `tag` | Image tag | ❌ | - |
| `version_label` | Value for app.kubernetes.io/version label | ❌ | - |
| `annotations` | Annotations to add (key:value or key=value format, one per line) | ❌ | - |
| `labels` | Labels to add (key:value or key=value format, one per line) | ❌ | - |
| `replicas` | Replicas count to set (JSON format) | ❌ | - |
| `configmap_values` | ConfigMap values to inject (JSON format) | ❌ | - |
| `namespace` | Set namespace | ❌ | - |
| `name_prefix` | Add name prefix | ❌ | - |
| `name_suffix` | Add name suffix | ❌ | - |

## Outputs

| Output | Description |
|--------|-------------|
| `overlay_dir` | Path to the edited overlay |
| `kustomization_file` | Path to kustomization.yaml |

## Examples

### Basic image update
```yaml
- name: Update image tag
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/staging
    image: myapp
    tag: ${{ github.sha }}
```

### With labels and annotations
```yaml
- name: Full update
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/production
    image: backend
    tag: v2.0.0
    labels: |
      version:v2.0.0
      environment:production
      team:platform
    annotations: |
      deployed-by:${{ github.actor }}
      deployment-id:${{ github.run_id }}
    # Both key:value and key=value formats are supported
```

### With debug output
```yaml
- name: Update with debugging
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/staging
    image: api
    tag: latest
    debug: 'true'  # Shows final kustomization.yaml
```

### Inject ConfigMap values
```yaml
- name: Deploy with runtime configuration
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/production
    image: backend
    tag: ${{ github.sha }}
    configmap_values: |
      [
        {"key": "SENTRY_RELEASE", "value": "${{ github.sha }}"},
        {"key": "DEPLOYMENT_ID", "value": "${{ github.run_id }}"},
        {"key": "ENVIRONMENT", "value": "production"}
      ]
```

This automatically finds the ConfigMap defined in your `configMapGenerator` and patches these values into it at deployment time. Perfect for injecting runtime values like release versions, deployment IDs, or environment-specific configuration.

### Update multiple images
```yaml
- name: Update API image
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/dev
    image: api
    tag: latest

- name: Update worker image
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/dev
    image: worker
    tag: latest
```

### With namespace override
```yaml
- name: Update for feature branch
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/feature
    image: app
    tag: feature-${{ github.event.number }}
    namespace: pr-${{ github.event.number }}
```

## How It Works

The action modifies `kustomization.yaml`:

### Before
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
images:
  - name: backend
    newTag: v1.0.0
```

### After
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
images:
  - name: backend
    newTag: v1.2.3
commonLabels:
  version: v1.2.3
commonAnnotations:
  last-deployed: "2024-01-15T10:30:00Z"
```

## Advanced Usage

### Working with different container registries

The action supports both simple image names and full registry paths. This is useful when:
- Your images are stored in different registries (Docker Hub, GCR, ECR, etc.)
- You need to specify exactly which registry to pull from
- You're using private or corporate registries

```yaml
# Simple image name (uses default registry)
- name: Update from Docker Hub
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/production
    image: nginx
    tag: 1.21

# Full registry path for Google Container Registry
- name: Update from GCR
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/production
    image: gcr.io/my-project/backend
    tag: v1.2.3

# Private registry example
- name: Update from private registry
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/production
    image: registry.company.com/team/api-service
    tag: ${{ github.sha }}
```

When you provide the full registry path, kustomize knows exactly which image entry to update in your kustomization.yaml, even if you have multiple images with the same base name from different registries.

## Notes

- Preserves existing kustomization.yaml structure
- Creates kustomization.yaml if it doesn't exist
- Supports both `newTag` and `newName` patterns
- Works with strategic merge patches
- Compatible with kubectl 1.14+