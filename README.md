# Install L5-media-manager
##Easy eloquent media manager for Laravel 5.1

# Demo:

#### Add
<img src="https://cloud.githubusercontent.com/assets/6444260/10274954/97aad228-6b43-11e5-865a-d770aba57530.png" style="vertical-align: text-top">

#### List
<img src="https://cloud.githubusercontent.com/assets/6444260/10274952/97a5f618-6b43-11e5-9fc2-c5e86000205c.png" style="vertical-align: text-top">

#### Info
<img src="https://cloud.githubusercontent.com/assets/6444260/10274953/97a890bc-6b43-11e5-9e1c-b1aec3623d92.png" style="vertical-align: text-top">


Edit your required dependencies in composer.json:
`  "joanvt/l5-media-manager": "dev-master"  `

####Step 1 - With CMD Composer execute:
`composer update`

####Step 2 - Then, with CMD PHP execute:
` php artisan vendor:publish `

####Step 3 - Now, just add the provider class in your config/app.php:

` Joanvt\MediaManager\MediaManagerServiceProvider::class `

####Step 4 - Migrate the database. With CMD PHP execute:
` php artisan migrate`

####Step 5 - Create your Media Model (You can use an Alias if you want difining it in config/app.php ):

```
namespace app\Models;

use Illuminate\Database\Eloquent\Model;
class MyMediaClass extends Model {

    protected $table = 'media';
    protected $fillable = ['name','path','description','ext','gallery_id','status'];
    protected $guarded = ['id'];
}
```

####Step 6 - Create your view with our three methodes and include your query in "lists" method:
```
      {!! Joanvt\MediaManager\MediaManager::styles() !!}

      {!!  Joanvt\MediaManager\MediaManager::lists(Media::where('status','A')->orderBy('id','DESC'))  !!}
   
    	{!! Joanvt\MediaManager\MediaManager::scripts() !!}
```


####Step 7 - Create a Request validator (app/Http/Requests):

``` 
<?php

namespace app\Http\Requests;

use app\Http\Requests\Request;

class MediaFilesRequest extends Request {

    public function authorize() {
        return true;
    }

    public function rules() {
    	
    	return [
			'upl'	=> 'required|image|mimes:jpeg,bmp,png,gif|max:2000', // 2 Megas
	
		];
    }

    public function messages() {
        return [
        	'upl.required'				=>	'You have to upload some file',
      		'upl.max'				      =>	'2MB max size',
      		'upl.mimes'					  =>	'Not a valid file'
        ];
    }
	
    
}

```


####Step 9 - Create your Ajax Controller like this and include your File Requester:
```
<?php

namespace app\Http\Controllers;

use Illuminate\Routing\UrlGenerator;
use app\Http\Requests\MediaFilesRequest;
use Config;
use File;
use Media;
use Image;

class AjaxController extends Controller {

    protected $url;

    // ------------------------------------------------------------------
    public function __construct(UrlGenerator $url) {

        $this->url = $url;
		$this->middleware('isAdmin');
		$this->middleware('isAjax');
    }

    public function index(MediaFilesRequest $request){

		
		$$file = $request->file('upl');
		
		if(!$file->isValid()){
			return abort(405);
		}
		
		if(!file_exists(public_path(Config::get('jmedia.upload_path')))){
			File::makeDirectory(public_path(Config::get('jmedia.upload_path')), 0777,true);
		}
		
		
		if(!file_exists(public_path(Config::get('jmedia.upload_path').'/'.Config::get('jmedia.thumbnail_directory')))){
			File::makeDirectory(public_path(Config::get('jmedia.upload_path').'/'.Config::get('jmedia.thumbnail_directory')), 0777,true);
		}
		
		$name = md5(uniqid()).$file->getClientOriginalName();
		$request->file('upl')->move(public_path(Config::get('jmedia.upload_path')), $name.'.'.$file->getClientOriginalExtension());
		
		
		
		Media::create([
			'name' 			=> $name,
			'path' 			=> Config::get('jmedia.upload_path'),
			'description'	=> '',
			'ext'			=> $file->getClientOriginalExtension(),
			'status'		=> 'A'
		]);
		
		$width  = Config::get('jmedia.width_thumbnail');
		$height = Config::get('jmedia.height_thumbnail');
		
		$image = Image::make(Config::get('jmedia.upload_path').'/'.$name.'.'.$file->getClientOriginalExtension());
		
		$path = public_path(Config::get('jmedia.upload_path').'/'.Config::get('jmedia.thumbnail_directory'));
		
		$image->resize($width,$height);
		$image->save($path.'/'.$name.$width.'x'.$height.'.'.$file->getClientOriginalExtension());
		
    }
    
}

```

####Step 10 - You can configure some parameters about the file uploading in config/jmedia.php:
```
 return [
	
	'upload_route'			=>	url('ajax/upload_files/'),
	'upload_path'			=>	'uploads/'.date('d-m-Y'),
	'thumbnail_directory'		=>	'thumbnails',
	'width_thumbnail'		=>	200,
	'height_thumbnail'		=>	200
	
 ];
```


##Tips:

####Use .htaccess for avoid the execution of some binary files  i.e: 
``php_flag engine off``

##### (Image Alias) Intervention dependency
`` We use "intervention/image": "2.*" dependency for image manipulation. You can also use native PHP  ``




#To-do List:

 ```
   -- Add Gallery (Table created but not implemented yet)
   -- Delete items
 ```

