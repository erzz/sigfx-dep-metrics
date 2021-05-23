# Signal FX Deployment Metrics

Quick and dirty pinging of deployment metrics to Signal FX.

## How to Use
Typically you would use the action to send one message to Signal FX to say the deployment has started with a status of "pending" and then once more after deployment to say the deployment has finished along with some sort of result ( `${{ steps.deploy.outcome }}` is really useful for this!)

- The action assumes `https://ingest.eu0.signalfx.com/v2/datapoint` is the host (configurable with the `host` input)
- The action requires a Signal FX API token to authenticate (using the `sigfx_token` input)
- If no application named it will use the name of the repository
- If no version specified it will use the short SHA of the commit
- If no environment names it will use the name of the branch
- Note that it is good to use `if: always()` for the post deployment execution so that it actually sends a result even if the deployment failed

## Examples

### Feature Branch would use mostly defaults

Assuming the:
* SIGFX token is stored in a secret named `SIGFX_ORG_TOKEN`
* SIGFX host = `https://ingest.eu0.signalfx.com/v2/datapoint`
* Service = rpim

```yaml
- name: Notify Deployment Start
  uses: erzz/sigfx-dep-metrics
  with:
    sigfx_token: ${{ secrets.SIGFX_ORG_TOKEN }}
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
    sigfx_token: ${{ secrets.SIGFX_ORG_TOKEN }}
    status: finished
    result: ${{ steps.deploy.outcome }}
```

### Staging we would override some details

```yaml
- name: Notify Deployment Start
  uses: erzz/sigfx-dep-metrics
  with:
    sigfx_token: ${{ secrets.SIGFX_ORG_TOKEN }}
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
    sigfx_token: ${{ secrets.SIGFX_ORG_TOKEN }}
    status: finished
    result: ${{ steps.deploy.outcome }}
    environment: staging
```

### Production we want to use tag for version

Here we are including the slugify action to get a nicely formatted version based on the git tag

```yaml
- name: Slugify github variables
  uses: rlespinasse/github-slug-action@v3.x

  ...
  
- name: Notify Deployment Start
  uses: erzz/sigfx-dep-metrics
  with:
    sigfx_token: ${{ secrets.SIGFX_ORG_TOKEN }}
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
    sigfx_token: ${{ secrets.SIGFX_ORG_TOKEN }}
    status: finished
    result: ${{ steps.deploy.outcome }}
    environment: production
    version: ${{ env.GITHUB_REF_SLUG_URL }}
```