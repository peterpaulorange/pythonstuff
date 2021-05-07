const request = require('request-promise');
const download = require ('node-image-downloader');
const cheerio = require('cheerio');
//const url = 'https://readbengalibooks.com/index.php/shop-by-categories/books/junior-classics.html?manufacturer=862';

var fs = require('fs');

(async function(){

	try{

		const url = 'https://readbengalibooks.com/index.php/shop-by-categories/books/horror.html?dir=asc&limit=75&order=name&p=2';


		const mainHtml = await request(url);
		const $ = cheerio.load(mainHtml);		
		

		const links = $('.item.last').map((i, section) => {
			const $link = $(section).find('.product-name a');
			return $link.attr('href');
			}).get();

		//console.log(links)


		await Promise.all(links.map(async (link) => {

			try {

				const productLink = await request(link);
				const $ = cheerio.load(productLink)

					const titles =  $('.product-name div').map ((i, elem) =>{

						const $title = $(elem).find('.details_heading_txt');
						return $title.text().split('\n');

					}).get();

					const prices = $('.price-info div').map ((i, elem) =>{

						const $price = $(elem).find('.old-price span');
						return $price.text().split('\n');

					}).get();

					const altPrice = $('.price-info div').map ((i, elem) =>{

						const $aPrice = $(elem).find('.regular-price span');
						return $aPrice.text();

					}).get();

					const category = $('.breadcrumbs ul').map ((i, elem) =>{

						const $cat = $(elem).find('.category a');
						return $cat.text();

					}).get();

					const images = $('.product-image div').map ((i, elem) =>{

						const $img = $(elem).find('.product-image-gallery img');
						return $img.attr('src')

					}).get();



					const features = $('.key-features-section div').map ((i, elem) =>{

						const $feat = $(elem).find('.other-section p');
						return $feat.text().split('\n')

					}).get();



						download({
								imgs: [
								{

									uri: images[0]
								}],

								dest: './horror'
						
								})

								.then ((info)=>{

								console.log("Download complete")

						

							})	

			


					await console.log(titles[0].trim() + ',' 
						+ titles[2].trim()+',' 
						+ prices[1].trim() 
						+ altPrice + ',' 
						+ category +','
						+ images + ','
						+ features[2].trim()+ ','
						+ features[8].trim())

					await fs.appendFile('horror.csv', titles[0].trim() + '/' 
						+ titles[2].trim()+'/'+'/'+'/'+'/'
						+ prices[1].trim() 
						+ altPrice + '/' 
						+ category +'/'
						+ images + '/'
						+ features[2].trim()+ '/'
						+ features[8].trim()+ '\n', function(err){
							if (err) throw err;
							console.log('Price is created');

						});

					}catch (e){
					console.log('my' + e);
				}


			}))

		} catch (e){
			console.log("Run"+ e);
	}

	console.log("\007");

})();
