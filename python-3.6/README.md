# redhat-codeready-dependency-analysis Python (3.6)  Action

A GitHub Action for using Red Hat CodeReady Dependency Analytics platform to detect vulnerabilities in your Python-3.6 projects.

You can use the Action as follows:

```yaml
name: Workflow using RedHat-CodeReady-Dependency-Analysis action
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: setup
        run: |
          mkdir -p site-dir
          pip3 install --target=site-dir -r requirements.txt      
      - name: redhat-codeready-dependency-analysis
        uses: fabric8-analytics/gh-actions/python-3.6@main
        with:
          manifest-file-path: requirements.txt
          output-file-path: redhat-codeready-dependency-analysis-report.json
          pkg-installation-directory-path: site-dir
        env:
          CRDA_KEY: ${{ secrets.CRDA_KEY }}
          CONSENT_TELEMETRY: ${{ secrets.CONSENT_TELEMETRY }}
      - name: post-action-setup
        run: |
          printf "Report file data:\n"
          cat redhat-codeready-dependency-analysis-report.json
```
