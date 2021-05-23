## Examples

### Feature Branch would use mostly defaults

Assuming the:
* SIGFX token is stored in a secret named `SIGFX_ORG_TOKEN`
* SIGFX host = `https://ingest.eu0.signalfx.com/v2/datapoint`
* Service = rpim

```yaml
- name: Slugify github variables
  uses: rlespinasse/github-slug-action@v3.x

  ...
  
- name: Notify Deployment Start
  uses: erzz/sigfx-dep-metrics
  with:
    status: started
    result: pending

# NOTE the id = deploy
- name: Deploy Environment
  id: deploy
  run: ./mydeployscript.sh

- name: Notify Deployment Success
  uses: erzz/sigfx-dep-metrics
  if: always()
  with:
    status: finished
    result: ${{ steps.deploy.outcome }}
```

### Staging we would override some details

```yaml
- name: Slugify github variables
  uses: rlespinasse/github-slug-action@v3.x

  ...

- name: Notify Deployment Start
  uses: erzz/sigfx-dep-metrics
  with:
    status: started
    result: pending
    environment: staging

# NOTE the id = deploy
- name: Deploy Environment
  id: deploy
  run: ./mydeployscript.sh

- name: Notify Deployment Success
  uses: erzz/sigfx-dep-metrics
  if: always()
  with:
    status: finished
    result: ${{ steps.deploy.outcome }}
    environment: staging
```

### Production we want to use tag for version

```yaml
- name: Slugify github variables
  uses: rlespinasse/github-slug-action@v3.x

  ...
  
- name: Notify Deployment Start
  uses: erzz/sigfx-dep-metrics
  with:
    status: started
    result: pending
    environment: production
    version: ${{ env.GITHUB_REF_SLUG_URL }}

# NOTE the id = deploy
- name: Deploy Environment
  id: deploy
  run: ./mydeployscript.sh

- name: Notify Deployment Success
  uses: erzz/sigfx-dep-metrics
  if: always()
  with:
    status: finished
    result: ${{ steps.deploy.outcome }}
    environment: production
    version: ${{ env.GITHUB_REF_SLUG_URL }}
```