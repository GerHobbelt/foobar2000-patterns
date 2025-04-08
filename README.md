# [foobar2000](https://www.foobar2000.org/) file / path naming [patterns](https://wiki.hydrogenaud.io/index.php?title=Foobar2000:Title_Formatting_Reference)

## File Management: File Move/Copy

*File Name Patterns* which I use most often:

### artist/album/CD/tracks with Windows path length clipping for MAX_PATH and anti-collision

#### Features:

- generated path is *guaranteed* to fit in the limited path length for general Windows files (260 characters ~ MAX_PATH)
- generated path includes hashes and index number to ensure (semi-)duplicate files don't overwrite one another

#### Pattern:

```
\!music\_\$trim($left(%album artist%,48) $iflonger(%album artist%,48,[~H[$pad($add($mod($crc32(%album artist%),97),1),2,X)]],[]))[\$trim($left(%album%,48) $iflonger(%album%,48,[~H[$pad($add($mod($crc32(%album%),97),1),2,X)]],[]))][\CD %disc number%]\$num(%tracknumber%,2). $trim($left(%title%,70) $iflonger(%title%,70,[~H[$pad($add($mod($crc32($trim(%title%)),97),1),2,X)]],[]))##$iflonger([$add(%length_seconds_fp%,0).$left($ext(%length_seconds_fp%),3)s],[%length_ex%],[%length_ex%],[$add(%length_seconds_fp%,0).$left($ext(%length_seconds_fp%),3)s])#$mod($add(%list_index%,41),97)
```

#### Explanation of the various parts of the pattern

* `\!music\_\` 

  This is the root path for all my (postprocessed) sound files.

  By having this part in the pattern rather than in the **Destination Folder** ensures that foobar2000 will *create* those directories automatically, instead of barfing a hairball about a paths not present.
  
* `$trim($left(%album artist%,48) $iflonger(%album artist%,48,[~H[$pad($add($mod($crc32(%album artist%),97),1),2,X)]],[]))`

  The 'album artist' first level of organization: I use the *album artist* instead of `%artist%` because the latter *can* differ per track within a single album for collaborations and collection albums ("Classics from the '70s", that sort of thing). Foobar2000 will automatically pick the [`%artist%`](https://wiki.hydrogenaud.io/index.php?title=Foobar2000:Title_Formatting_Reference#.25artist.25) entry when the [`%album arist%`](https://wiki.hydrogenaud.io/index.php?title=Foobar2000:Title_Formatting_Reference#.25album_artist.25) entry in the music track properties is *empty*, so this will never be empty if you have properly filled in the Artist property entry at least.
  
  Now about the parts in this one:
  
  + `$trim(` - meant to remove trialing whitespace, which will occur when the last part (the 'hash' section, see below) delivers an empty part string.
  
  + `  $left(%album artist%,48)` - clips the Album Artist text length to 48 characters.
  
    > Yes, the 48 was an arbitrary choice: I've got 260 characters for the entire path, which I have to distribute among the base, artist, album, CD and track parts of this path pattern. The titles generally are longer and deemed more important by me, so I have reserved more space for that part, while artist and album only get about 48 space allocated to them. Tweak the numbers if you want a different 'distribution'.
    
  + `  $iflonger(%album artist%,48,  {HASHID}  ,[]))` - see [the $iflonger documentation](https://wiki.hydrogenaud.io/index.php?title=Foobar2000:Title_Formatting_Reference#.24iflonger.28str.2Cn.2Cthen.2Celse.29): here I create a 'unique hash string' *only when the Album Artist text happens to be clipped by the preceding `$left(...)` operation*.
  
  + `    [~H[$pad($add($mod($crc32(%album artist%),97),1),2,X)]]` - create a 'unique hash string' for the given Album Artist. whoa, Nelly! What is this?!
  
    - I wanted it to be easily recognizable to me, hence the `~H` prefix
    
    - we only use the Album Artist as a *seed* for the hash calculation to ensure the hash is always the same for every track listing this particular Album Artist: after all, we want all those tracks to land *inside* this single directory identifying the Album Artist, whether it is "V.A." (short for "Various Artists") or a (very) long Symphonic Orchestra + Conductor identifier for some classical music albums I've got. (*Those* were the ones that started all this as the usual `%album artist%/%album%/%title%` resulted in filepaths which were waaaay too long and caused a lot of trouble.
    
    - `$crc32(%album artist%)` - calculate the CRC32 hash of the Album Artist text. We use this number as the base of our hash calculation as it produces a large-ish number which is close enough to 'semi-random' to satisfy the requirement that different Album Artists will produce different CRC32 values.
    
    - `$mod( {CRC32} ,97)` - then we shorten that number to 2 decimal digits by taking the remainder MODULO a prime: this will keep the 'semi-randomness' in the original CRC32 number as much as possible. 'modulo prime' is an operation often used in fast (semi-)random generators used in regular software. It's certainly not Secure Random or anything, but it's good enough for us here.
    
    - `$add( {MOD97} ,1)` - the modulo operation will have delivered a number between 0 and 96. I didn't like a zero hash value, no reason, just taste, so I add 1, which shiftsthe output number range to 1-98, which is still 2 decimal digits max.
    
    - `$pad( {NUMBER} ,2,X)` - here we 'print' the number to become a string. When it only takes a single digit, we 'pad' with 'X'; I used an 'X' instead of a zero or other digit, so that the hash values 1-9 would be different from 10, 20, 30, etc. as the padding happens at the tail, rather than the head. 
    
      Anyway, this little extra thing ensures I *always* have a hash 'identifier' which is exactly 2 characters wide. Add that to the hash prefix (`~H`, 2 characters) and the worst-case Album Artist text length is 48+2+2=52 characters.
      
    - `[~H[ {HASHID} ]]` - this prefixes the hash with the `~H` prefix. The outermost `[]` brackets are not necessary but helped me delimit and thus identify this complex part of an even more complicated File Naming Pattern.

* `[\$trim($left(%album%,48) $iflonger(%album%,48,[~H[$pad($add($mod($crc32(%album%),97),1),2,X)]],[]))]`

  Create the Album directory. As with the Album Artist, we limit this one to 48 characters. The process is exactly the same, only this time, of course, we use the *Album* as the seed for the hash.
  
  See above at the Album Artist part how the hash is constructed exactly.

  + `[\ ... ]` - only create this directory if there actually *is* an Album title.
  
  + `$trim($left(%album%,48)` - clip the Album title to 48 characters tops.
  
  + `$iflonger(%album%,48,[~H[$pad($add($mod($crc32(%album%),97),1),2,X)]],[]))` - construct the `~H42`-lookalike hash postfix when the album title happens to have been clipped.

* `[\CD %disc number%]`

  Create a `CD 1`, etc. subdirectory when the metadata (track properties) mention that the album is a multi-CD (or multi-LP) release.
  
  > Note: of course I made a mistake a few times and forgot to enter the CD number in the properties. The hashes at least helped to ensure no files would be overwritten and then I had to go in and individually edit the track properties to ensure every track was assigned the correct CD number. Taught me a lesson not to make this sort of mistake too often!
  
* `\$num(%tracknumber%,2). $trim($left(%title%,70) $iflonger(%title%,70,[~H[$pad($add($mod($crc32($trim(%title%)),97),1),2,X)]],[]))##$iflonger([$add(%length_seconds_fp%,0).$left($ext(%length_seconds_fp%),3)s],[%length_ex%],[%length_ex%],[$add(%length_seconds_fp%,0).$left($ext(%length_seconds_fp%),3)s])#$mod($add(%list_index%,41),97)`

  The track filename formatted as 'TrackNumber. Title#UniqueID' where the 'UniqueID' part is constructed from both the play length in milliseconds and a semi-arbitrary number: the index in the current playlist. That last bit will ensure that even *exact duplicates* will not overwrite the other when moved: I use different software for that task and don't foobar2000 to nuke a potentially good track by overwriting it with a mediocre sampling or ditto encoding of the same.
  
  The parts explained:
  
  + `\$num(%tracknumber%,2).` - start with the track number, padded to 2 digits, so the default sort order, which is *text*-based rather than *numeric*, sorts the tracks in the order they appeared on the album. Append a dot and a space. That's a matter of taste: before I've used a space-dash-space separator but the dot-space one is 1 character shorter, so that's 1 more character in which we may encode actually *useful* info in the filepath. If you don't like it, edit and fill in your own poison. :wink:
  
  + `$trim(  {TITLE}  )` - again a `$trim` to strip off any trailing whitespace that will be created by the inner pattern for any Album title that's shorter than or equal to 70 characters.
  
  + `  $left(%title%,70)` - *l'histoire ce repête*: clip Track Ttitle to 70 characters, but otherwise this is the same as what we did for Album Artist and Album title.
  
  + `  $iflonger(%title%,70,  {HASHID}  ,[])` - same story here: only inject the generated 'unique hash' when we actually did clip the title.
  
  + `    [~H[$pad($add($mod($crc32($trim(%title%)),97),1),2,X)]]` - see further above for the details of this one. This time we're using the Track Title as the *seed* as that is what we're shortening this time.
  
  + `  ##` - just a separator that I picked to be pretty unique iff I ever have to machine-parse the filenames/patchs ever again. Besides, it helps humans like me to identify the extra bits in the track title filename more easily as there *are* Track Titles out there that are only numbers, etc. which could otherwise be easily confused with what follows this bit:
  
  + `  $iflonger([$add(%length_seconds_fp%,0).$left($ext(%length_seconds_fp%),3)s],[%length_ex%],[%length_ex%],[$add(%length_seconds_fp%,0).$left($ext(%length_seconds_fp%),3)s])` 
  
    This long blurb encodes the play time of the track either as a foobar2000-default HMSm time format *or* as the number of seconds, whichever one is the shorter one, where the playtime-in-seconds is printed as a floating point number that's restricted to 3 digits past the decimal dot, so we can glean the number of partial seconds, i.e. *milliseconds* in there.
    
    The hairy bit is shortening the foobar2000 `%length_seconds_fp%` floating point value to 3 digits past the decimal dot as the numeric operations offered by foobar2000 are integer based.
    
    Hence we take that floating point number and 'split' into an integer and fractional part: taking the integer part is easy: use *any* numeric operation which does not alter the value, e.g. add *zero* and you're golden: `$add(%length_seconds_fp%,0)`. 
    
    Extracting the fractional is a little harder as we don't a numeric command for that, nor is there a string-split command either, but there *is* a command which recognizes a (decimal) dot and can deliver what comes after it: `$ext()` treats the floating point number as a filename with extension and delivers the extension.
    *Now* we have a second problem: the fractional part won't always be 7 digits as a play length *could* be, for example, 240.15 seconds, hence `$ext(240.15)` would spit out `15`, so we must reckon with that, despite the rarity of this happening, at least in my play list. So we settle for 'rounding' that number to 3 digits by taking the 3 *most siginificant* digits from there by way of `$left([value],3)`. And then we postfix a 's' *seconds* unit and presto!
    
    So that gives us the above pattern part, unfolded into these fragments:
    
    ```
    $iflonger(
      [
        $add(%length_seconds_fp%,0)
        .
        $left(
          $ext(%length_seconds_fp%) ,
          3)
        s
      ] ,
      [%length_ex%] ,
      [%length_ex%] ,
      [
        $add(%length_seconds_fp%,0)
        .
        $left(
          $ext(%length_seconds_fp%) ,
          3
        )
        s
      ]
    )
    ``` 

 ## New macros as per 2025AD

 ### Copy / move macros

 - favortites: destination path + `$if([%disc number%],$num(%disc number%,2):,)$if([%track number%],$num(%track number%,3),999$num(%list_index%,5)) - $shortest(%artist%,$left(%artist%,60)⋯) :: $shortest(%title%,$left(%title%,80)⋯)[ :: $shortest([%album%],$left([%album%],40)⋯)]##$num(%list_index%,5)`
 - AAS100: album-artist-title-clipped-to-below-260-chars: destination path + `$if([%disc number%],$num(%disc number%,2):,)$if([%track number%],$num(%track number%,3),999$num(%list_index%,5)) - $shortest(%artist%,$left(%artist%,60)⋯) :: $shortest(%title%,$left(%title%,80)⋯)[ :: $shortest([%album%],$left([%album%],40)⋯)]##$num(%list_index%,5)`
 - AAS100-2: same as above, but creates yet another subdirectory tree: phase 2 of the dir tree cleanup process: destination_path + `$if([%disc number%],$num(%disc number%,2):,)$if([%track number%],$num(%track number%,3),999$num(%list_index%,5)) - $shortest(%artist%,$left(%artist%,60)⋯) :: $shortest(%title%,$left(%title%,80)⋯)[ :: $shortest([%album%],$left([%album%],40)⋯)]##$num(%list_index%,5)`

### Rename macros

- "#D:#T-artist-track-clipped": `$if([%disc number%],$num(%disc number%,2):,)$if([%track number%],$num(%track number%,3),999$num(%list_index%,5)) - $shortest(%artist%,$left(%artist%,60)⋯) - $shortest(%title%,$left(%title%,100)⋯)##$num(%list_index%,5)`
- "favorites": `[$left(%date_added_to_pref_collection%,19) ~ ]$if([%album artist%],$shortest(%album artist%,$left(%album artist%,40)⋯),$shortest(%artist%,$left(%artist%,40)⋯)) :: [$shortest([%album%],$left([%album%],40)⋯) :: ]$if([%disc number%],$num(%disc number%,2):,)$if([%track number%],$num(%track number%,3),999$num(%list_index%,5)) - $shortest(%title%,$left(%title%,70)⋯)##$num(%list_index%,5)`




