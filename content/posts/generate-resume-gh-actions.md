---
title: "Generate Your Resume with GitHub Actions"
date: 2020-08-18T10:39:19-05:00
draft: false
---

I got tired of editing my resume in HTML and then printing a PDF from Chrome. I now use GitHub Actions and a json resume to generate both formats.

<!--more-->

## Defining a JSON Resume
There's a jsonresume project that defines a [JSON Schema](https://json-schema.org/) for a resume. You can find the schema repo [here](https://github.com/jsonresume/resume-schema).

I wanted to define my resume like this so I could easily generate multiple formats of it. My file can be found [here](https://github.com/kasuboski/resume/blob/master/resume.json). I used to use the [resume-cli](https://github.com/jsonresume/resume-cli) project to generate the html and pdf version of my resume, but it stopped working for me awhile ago.

I decided to convert the theme I was using to a Go template instead. That template is [here](https://github.com/kasuboski/resume/blob/master/hack/resume.html.tmpl). It treats `resume.json` as a map so the template just directly accesses the properties.

```golang
// hack/template.go
tmpl := template.Must(template.ParseFiles("hack/resume.html.tmpl"))
bs, err := ioutil.ReadFile("resume.json")
if err != nil {
  log.Fatalf("couldn't read resume.json: %v", err)
}

f, err := os.Create("resume.html")
if err != nil {
  log.Fatalf("couldn't open out.html: %v", err)
}
defer f.Close()

var params map[string]interface{}
err = json.Unmarshal(bs, &params)
if err != nil {
  log.Fatalf("unable to unmarshal json: %v", err)
}

tmpl.Execute(f, params)
```

Now I can get the HTML version of my resume with `go run hack/template.go` which will output a `resume.html` file. I could then open this in chrome and print it from there, but that's so much effort :wink:.

## Generating Multiple Formats in GitHub Actions
I already had a GitHub Actions workflow that would sync my resume from the resume repo to my personal-site repo, but I was manually pushing the HTML and PDF files to the resume repo. Now that I have the HTML generation working again, I decided to automate the entire process. That includes creating GitHub Releases.

My resume repo [kasuboski/resume](https://github.com/kasuboski/resume) now has a `create-release` workflow. Pushing changes to `resume.json` or tagging a commit will now do the below.

* Generate the html with `go run hack/template.go`
* Generate a PDF using [fifsky/html-to-pdf-action](https://github.com/fifsky/html-to-pdf-action)
* Add the html and pdf as a build artifact (so you can manually inspect before releasing)
* Create a release for a tag with the files
* Update my personal site with the new files if they've changed

Updating my resume now involves just updating `resume.json` and the files are generated and pushed to my site.

## Future
I'd like to start managing more things like this. Maybe pushing to my LinkedIn profile or updating a GitHub profile README. It'll be like [GitOps](https://www.weave.works/technologies/gitops/) but more Git...personal info.