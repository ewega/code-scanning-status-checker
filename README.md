# Code Scanning Status Checker

The Actions is meant to help you if you're using both GitHub Merge Queue and GitHub Advanced Security. It is a workaround to functionality that will be native to GitHub at some point.

At the time of creation of this Action, the ability to do **all** of the following is **not** possible:
1. Have Merge Queue turned on in a repo
2. Have Code Scanning from GitHub Advanced Security activated on a repo
3. Enforce Code Scanning's CodeQL as a [required status check](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/about-status-checks) for pull requests.

This is a very simple Action that will act as a substitute required status check for Code Scanning once you have merge queue enabled. It will force Code Scanning to pass at the Pull Request and allow you to skip it in your repo's merge group.

**Note:** Today this is configured to work only with the CodeQL engine that comes with GitHub Advanced Security.

## Why?

If you want more details as to why this workaround exists see [here](https://ewega.github.io/site/github/tutorial/2023/12/04/enabling-ghas-merge-queue/). 

## How does it work?

The action uses the GraphQL API to call the Status Checks API. It grabs the status of the CodeQL analysis (which should have already been run) in a PR. If the CodeQL status check fails, this action will fail, acting as a security gate.

## How to use?
1. Add it in a separate job to your workflow:

  ````yaml
  check_codeql_status:
    name: Check CodeQL Status
    needs: analyze
    permissions: 
      contents: read
      checks: read
      pull-requests: read
    runs-on: ubuntu-latest
    if: ${{ github.event_name == 'pull_request' }}
    steps:
    - name: Check CodeQL Status
      uses: eldrick19/code-scanning-status-checker@v2
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
        pr_number: ${{ github.event.pull_request.number }}
        repo: ${{ github.repository }}
  ````

  <details>
  <summary>For example, your workflow could look like:</summary>
  <br/>
    
  ````yaml
  name: "CodeQL"
  
  on:
    pull_request:
      branches: [ "main" ]
    merge_group:
  
  jobs:
    analyze:
      name: Analyze
      runs-on: ${{ (matrix.language == 'swift' && 'macos-latest') || 'ubuntu-latest' }}
      timeout-minutes: ${{ (matrix.language == 'swift' && 120) || 360 }}
      permissions:
        actions: read
        contents: read
        security-events: write
  
      strategy:
        fail-fast: false
        matrix:
          language: [ 'javascript-typescript' ]
  
      steps:
      - name: Checkout repository
        uses: actions/checkout@v3
  
      # Initializes the CodeQL tools for scanning.
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: ${{ matrix.language }}
  
      - name: Autobuild
        uses: github/codeql-action/autobuild@v2
  
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
        with:
          category: "/language:${{matrix.language}}"
  
     check_codeql_status:
        name: Check CodeQL Status
        needs: analyze
        permissions: 
          contents: read
          checks: read
          pull-requests: read
        runs-on: ubuntu-latest
        if: ${{ github.event_name == 'pull_request' }}
        steps:
        - name: Check CodeQL Status
          uses: eldrick19/code-scanning-status-checker@v2
          with:
            token: ${{ secrets.GITHUB_TOKEN }}
            pr_number: ${{ github.event.pull_request.number }}
            repo: ${{ github.repository }}
  ````
  </details>

2. Make the "Force CodeQL Check" job required in your branch protection settings. Your Advanced Security required status checks should look like:

    <img width="752" alt="Capture d’écran, le 2023-12-04 à 21 36 01" src="https://github.com/Eldrick19/code-scanning-status-checker/assets/26189114/06337b7a-1178-49a9-9990-fbd024f8a4e4">

3. If "CodeQL" was previously a required check, do not require it anymore

## How to test?

Install the necessary dependencies by running the following command in your project directory:
  ```
  npm install
  ```

Run the tests by executing the following command:
  ```
  npm test
  ```
