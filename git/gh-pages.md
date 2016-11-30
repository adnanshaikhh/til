# How to set up your repo with Github pages

## Create a ```CNAME``` in your repository
Create a ```CNAME``` in your repository with the domain or sub-domain you'd like to
reach the repo with.
Eg: adnanshaikh.com or blog.adnanshaikh.com

## Add a ```CNAME``` record for your domain
Go to your domain management settings and add a ```CNAME``` record with the
following values

_In case of a sub-domain_  
Add a ```CNAME``` record
* Name = ```blog```  
* Type = ```CNAME```  
* Value = ```adnanshaikhh.github.io``` (Add a dot in the end if it doesn't work)

_In case of a domain_  
Add a ```A``` record
* Name = (blank)
* Type = ```A```
* Value = ```192.30.252.153``` and ```192.30.252.154``` (Both are for Github)

Add a ```CNAME``` record  
* Name = ```www```  
* Type = ```CNAME```  
* Value = ```adnanshaikhh.github.io``` (Add a dot in the end if it doesn't work)

Because of the way DNS records are cached across the internet, these sorts of
changes can take a few hours to take effect.

## Create a ```gh-pages``` branch
If you already have a gh-pages branch, make sure it's synced up to
master, showing exactly what you want. If you don't, type:

```
git checkout -b gh-pages
git push origin gh-pages
```

Second, go to GitHub, settings for your repo, and switch “Default Branch”
to gh-pages.

Third, switch to your gh-pages branch, and delete the local master branch with

```
git branch -d master
```

Finally, delete the remote ```master``` branch with

```
git push origin :master
```

Done. You now have only the gh-pages branch, and it's the default one that git
will use when talking with GitHub.
