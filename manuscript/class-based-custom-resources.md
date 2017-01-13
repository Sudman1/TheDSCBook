# Class-Based Custom Resources
Class-based resources were introduced in WMF5. They offer a few core differences over function-based resources:

* They don't require a schema MOF.
* The class structure is enforced by PowerShell, meaning you have to try harder to mess up the Get-, Set-, and Test- names.
* In a class, the **return** keyword writes something to the output pipeline (just like Write or Write-Output), and then immediately exits the function (outside of a class, **return** is essentially an alias to **Write** and does not immediately exit).

If you've put most of your code into a standalone script module, as I recommended in the previous chapter and will elaborate on in the next chapter, then switching from a function-based resource to a class-based resource is easy. In fact, all I'm going to do in this chapter is "translate" what I did in the previous chapter.

However, there's a downside to class-based modules at this time, which is that they don't support filename-based versioning. This makes them a little problematic in real-world applications, because you can't have two versions of a class-based module living side-by-side on nodes or on a pull server. 

## Writing the Class-Based Interface Module
Here we go: a class-based version of the previous chapter's resource module.

```
[DscResource()]
Class FileContent {
	[DscProperty(Key)]
	[string]$Path
	
	[DscProperty(Mandatory)]
	[string]$DesiredContent
	
	[FileContent] Get() {
	    If (Test-Path $Path) {
        	$content = Get-Content -Path $path | Out-String
    	} else {
      	  	$content = $null
    	}
		
		$this.DesiredContent = $content
		$this.Path = $Path
		return $this
	
	}
	
	[void] Set() {
		Set-Content -Path $this.Path -Content $this.DesiredContent
	}
	
	[bool] Test() {
		return Test-FileContent -Path $this.path -DesiredContent $this.DesiredContent
	}
}
```

The Get method, replacing Get-TargetResource, is the one to look at. Notice that, above it, we're defining the parameters for this resource. They don't go into a Param() block, and I have to explicitly define the key property (which was done in the schema MOF of the function-based resource, rather than in the function itself). Rather than constructing a hashtable to output, I use the special $this object. $this has properties which match the DscProperty items defined at the top of the class - so, in this case, it has a Path and DesiredContent property. I set those, and then return the $this object. The Get() method is declared as returning an object of the type FileContent, which is the name of the class I created.

The Set() method returns nothing (the [void] part), and accesses the $this object to get the input parameters. Here, I just went with the Set-Content command rather than my pointless Set-FileContent "wrapper" function.

The Test() method returns True or False (that is, Boolean, or [bool]), and is simply running the Test-FileContent function I wrote previously. Again, it uses $this to access the input parameters.

As you can see, if you're keeping your resources simple in terms of functionality, and relying on functionality written elsewhere (either in PowerShell commands or functions that you've written in a normal script module), then deciding on a class-based or function-based resource is easy. They're very similar, once you really strip down all the code they contain.

## Preparing the Module for Use
Class-based resource modules get laid more simply than a function-based one:

/Modules/MyModuleName/MyModuleName.psm1
/Modules/MyModuleName/MyModuleName.psd1

There's no "DSCResources" subfolder hierarchy, in this case. That means you can't distribute your separate, "functional" script module in quite the same way. In this case, you'd simply drop it into the same folder, perhaps naming it something like MyModuleName-Support.psm1 or something. Your class-based module (MyModuleName.psm1) would need to explicitly load it, or your manifest (the .psd1 file) would need to specify it as an additional module to load along with the MyModuleName.psm1 root module.
