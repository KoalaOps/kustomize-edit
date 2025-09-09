# Kustomize Edit

Updates kustomize overlays with new image tags, labels, and annotations.

## Features

- 🏷️ **Image updates** - Set new image tags
- 📝 **Label management** - Add/update labels
- 🔖 **Annotations** - Set deployment metadata
- 📅 **Automatic timestamps** - Track deployment times
- 🎯 **Smart detection** - Handles various kustomize patterns

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
| `image` | Image name (without tag) | ❌ | - |
| `tag` | Image tag | ❌ | - |
| `version_label` | Value for app.kubernetes.io/version label | ❌ | - |
| `annotations` | Annotations to add (key=value format, one per line) | ❌ | - |
| `labels` | Labels to add (key=value format, one per line) | ❌ | - |
| `replicas` | Replicas count to set (JSON format) | ❌ | - |
| `json_patches` | JSON6902 patches to apply (JSON array) | ❌ | - |
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
      version=v2.0.0
      environment=production
      team=platform
    annotations: |
      deployed-by=${{ github.actor }}
      deployment-id=${{ github.run_id }}
```

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


### Multi-registry support
```yaml
- name: Update with full image path
  uses: KoalaOps/kustomize-edit@v1
  with:
    overlay_dir: deploy/overlays/production
    image: gcr.io/project/backend
    tag: v1.2.3
```

## Notes

- Preserves existing kustomization.yaml structure
- Creates kustomization.yaml if it doesn't exist
- Supports both `newTag` and `newName` patterns
- Works with strategic merge patches
- Compatible with kubectl 1.14+