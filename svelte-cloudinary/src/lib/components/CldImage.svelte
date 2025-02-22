<script lang="ts" context="module">
	export type CldImageProps = ImageOptions &
		Omit<HTMLImageElement, 'srcset' | 'height' | 'width'> & {
			src: string;
			width: string;
			height: string;
			alt: string;
			preserverTransformations?: boolean;
			// From Unpic
			layout?: 'constrained' | 'fullWidth' | 'fixed';
			priority?: boolean;
			background?: 'auto' | string;
		};
	import type { ImageOptions } from '@cloudinary-util/url-loader';
</script>

<script lang="ts">
	import { getTransformations } from '@cloudinary-util/util';
	import { transformationPlugins } from '@cloudinary-util/url-loader';
	import { Image } from '@unpic/svelte';
	import { getCldImageUrl } from '$lib/helpers/getCldImageUrl.js';

	/** 
	* set the compomnent $$props to be of type CldImageProps 
	*/
	type $$Props = CldImageProps

	const CLD_OPTIONS = ['deliveryType', 'preserveTransformations'];
	
	// reactively destructure the props
	$: ({
		alt,
		src,
		width,
		height,
		layout = 'constrained',
		priority = false,
		background = 'auto'
	} = $$props as $$Props);

	transformationPlugins.forEach(({ props = [] }) => {
		props.forEach((prop) => {
			if (CLD_OPTIONS.includes(prop)) {
				throw new Error(`Option ${prop} already exists!`);
			}
			CLD_OPTIONS.push(prop);
		});
	});
	const imageProps = {
		alt,
		src,
		width,
		height
	} as ImageOptions & $$Props;

	(Object.keys($$props) as Array<keyof $$Props>)
		.filter((key) => !CLD_OPTIONS.includes(key))
		// @ts-expect-error imageProps doesn't know the types of the keys
		.forEach((key) => (imageProps[key] = $$props[key]));

	// Construct Cloudinary-specific props by looking for values for any of the supported prop keys
	const cldOptions = {};
	CLD_OPTIONS.forEach((key) => {
		if ($$props[key]) {
			// @ts-expect-error cldOptions doesn't know the types of the keys
			cldOptions[key] = $$props[key] || undefined;
		}
	});

	// Try to preserve the original transformations from the Cloudinary URL passed in
	// to the component. This only works if the URL has a version number on it and otherwise
	// will fail to load
	if ($$props.preserveTransformations) {
		try {
			const transformations = getTransformations(imageProps.src).map((t) => t.join(','));
			// @ts-expect-error rawTranformations key is not declared in cldOptions
			cldOptions.rawTransformations = [
				...transformations.flat(),
				...($$props.rawTransformations || [])
			];
		} catch (e) {
			console.warn(`Failed to preserve transformations: ${(e as Error).message}`);
		}
	}
	let newSrc = getCldImageUrl({ ...imageProps, ...cldOptions });
</script>
<Image src={newSrc} {width} {height} {alt} {layout} {background} {priority} />
