<script>
import { currency, date } from '$lib/utils'
import { LazyImg } from '$lib/components'
import { onMount } from 'svelte'
import { page } from '$app/stores'
import { PrimaryButton, BackButton } from '$lib/ui'
import dayjs from 'dayjs'
import noAddToCartAnimate from '$lib/assets/no/add-to-cart-animate.svg'
import OrderListSkeleton from '../_OrderListSkeleton.svelte'
import OrderTracking from '../_OrderTracking.svelte'
import productNonVeg from '$lib/assets/product/non-veg.png'
import productVeg from '$lib/assets/product/veg.png'
import ReturnTracking from '../_ReturnTracking.svelte'
import SEO from '$lib/components/SEO/index.svelte'
import TransparentButton from '../_TransparentButton.svelte'

export let data
// console.log('zzzzzzzzzzzzzzzzzz', data)

let clazz
export { clazz as class }

const seoProps = {
	title: 'Order Details ',
	metaDescription: 'Order Details '
}

// let deliveryBy = null
let now = null
let selectedProduct = null
let showDemoScheduler = false
let loading = false

function head() {
	return {
		title: 'Order Details'
	}
}

onMount(() => {
	// deliveryBy = dayjs().add(7, 'day').format('dddd, MMM D, YYYY')
	now = dayjs()
})
</script>

<SEO {...seoProps} />

<div class="{clazz}">
	{#if loading}
		<OrderListSkeleton />
	{:else if data.order}
		<section>
			<BackButton to="/my/orders?sort=-updatedAt" class="mb-2" />

			<div class="mb-5 overflow-hidden rounded border sm:mb-10">
				<div class="flex flex-wrap items-center justify-between border-b bg-zinc-100 px-5 py-3">
					<h6>Order No : #{data.order?.orderNo}</h6>

					<h6>Order Date : {date(data.order?.createdAt)}</h6>
				</div>

				<!-- Order detail  -->

				<div class="grid grid-cols-1 divide-y lg:grid-cols-2 lg:divide-y-0 lg:divide-x">
					<div class="col-span-1 flex flex-col divide-y">
						{#if data.order?.items}
							<div class="divide-y divide-dashed text-xs text-zinc-500">
								{#each data.order?.items as item}
									<div class="flex gap-2 p-5 lg:gap-5">
										<a
											href="{`/product/${item.slug}`}"
											aria-label="Click to view the product details"
											class="shrink-0">
											<LazyImg
												src="{item.isCustomized ? item.customizedImg : item.img}"
												alt=""
												width="56"
												class="h-auto w-14 object-contain object-top" />
										</a>

										<div class="flex w-full flex-1 flex-col gap-0.5 xl:pr-4">
											<div class="flex justify-between gap-2 sm:gap-4">
												<a
													href="{`/product/${item.slug}`}"
													aria-label="Click to view the product details"
													class="flex-1 hover:underline">
													<p>
														{item.name}
													</p>
												</a>

												{#if $page?.data?.store?.isFnb && item.foodType}
													<div>
														{#if item.foodType === 'veg'}
															<img src="{productVeg}" alt="veg" class="h-5 w-5" />
														{:else if item.foodType === 'nonveg'}
															<img src="{productNonVeg}" alt="non veg" class="h-5 w-5" />
														{/if}
													</div>
												{/if}
											</div>

											{#if item.qty}
												<span>
													Qty :

													{item.qty}
												</span>
											{/if}

											{#if item.size}
												<span>
													Size :

													{item.size}
												</span>
											{/if}

											{#if item?.usedOptions?.length}
												{#each item?.usedOptions as option}
													{#if option?.val?.length && option?.val !== undefined && option?.val != ''}
														<div class="flex flex-wrap gap-2">
															<h6>{option.name}:</h6>

															{#if option.val}
																{#each option.val as v}
																	{#if v}
																		<div class="font-bold">
																			{v}
																		</div>
																	{/if}
																{/each}
															{/if}
														</div>
													{/if}
												{/each}
											{/if}

											<div class="flex flex-wrap items-center gap-1">
												Item price :

												<span class="font-bold whitespace-nowrap text-zinc-800">
													{currency(item.price, $page.data?.store?.currencySymbol)}
												</span>

												{#if item?.mrp > item?.price}
													<span class="whitespace-nowrap text-zinc-500 line-through">
														<strike>
															{currency(item.mrp, $page.data?.store?.currencySymbol)}
														</strike>
													</span>

													{#if Math.floor(((item.mrp - item.price) / item.mrp) * 100) > 0}
														<span class="whitespace-nowrap text-secondary-500">
															({Math.floor(((item.mrp - item.price) / item.mrp) * 100)}% off)
														</span>
													{/if}
												{/if}
											</div>

											<div class="flex flex-wrap items-center gap-1">
												Sub Total :

												<span class="font-bold whitespace-nowrap text-zinc-800">
													{currency(item.subtotal, $page.data?.store?.currencySymbol)}
												</span>

												{#if item?.total > item?.subtotal}
													<span class="whitespace-nowrap text-zinc-500 line-through">
														<strike>
															{currency(item.total, $page.data?.store?.currencySymbol)}
														</strike>
													</span>

													{#if Math.floor(((item.total - item.subtotal) / item.total) * 100) > 0}
														<span class="whitespace-nowrap text-secondary-500">
															({Math.floor(((item.total - item.subtotal) / item.total) * 100)}% off)
														</span>
													{/if}
												{/if}
											</div>

											{#if item?.status === 'delivered'}
												<div class="mt-2 xl:mt-0 xl:w-1/3">
													<a
														href="/my/reviews/create?pid={item?.pid}&oid={item?.orderItemId}&ref=/product/{item?.slug}"
														aria-label="Click to visit rate & review product"
														class="whitespace-nowrap font-semibold text-indigo-500 focus:outline-none hover:underline">
														Rate & Review Product
													</a>
												</div>
											{/if}
										</div>
									</div>
								{/each}
							</div>

							<div class="p-5 flex justify-end">
								<div class="flex max-w-max flex-col items-start gap-2">
									<p class="flex items-center">
										<span class="mr-2 w-32">Subtotal</span>

										<span>
											: &nbsp; {currency(
												data.order?.amount.subtotal,
												$page.data?.store?.currencySymbol
											)}
										</span>
									</p>

									<p class="flex items-center">
										<span class="mr-2 w-32">Discount</span>

										<span>
											: &nbsp;

											{currency(data.order?.amount.discount, $page.data?.store?.currencySymbol)}
										</span>
									</p>

									{#if data.order?.coupon.code}
										<p class="flex items-center">
											<span class="mr-2 w-32">Applied Coupon</span>

											<span>
												: &nbsp;
												{data.order?.coupon.code}
											</span>
										</p>
									{/if}

									<p class="flex items-center">
										<span class="mr-2 w-32">Shipping</span>

										<span>
											: &nbsp;
											{#if data.order?.amount.shipping}
												{currency(data.order?.amount.shipping, $page.data?.store?.currencySymbol)}
											{:else}
												Free
											{/if}
										</span>
									</p>

									<hr class="w-full border-t border-zinc-200" />

									<div class="flex items-center text-sm font-bold text-zinc-800">
										<span class="mr-2 w-32">Total</span>

										<span>
											: &nbsp; {currency(
												data.order?.amount.total,
												$page.data?.store?.currencySymbol
											)}
										</span>
									</div>
								</div>
							</div>
						{:else}
							<p>No order items found</p>
						{/if}
					</div>

					<div class="col-span-1 flex flex-col gap-5 p-5 lg:gap-10">
						<div>
							<h5 class="mb-2">Delivery Address</h5>

							<p class="flex flex-col">
								<span>
									{data.order?.address?.firstName}
									{data.order?.address?.lastName}

									<br />

									{data.order?.address?.address}, {data.order?.address?.city},
									{data.order?.address?.country}, {data.order?.address?.state}

									<br />

									{data.order?.address?.zip}
								</span>
							</p>

							{#if data.order?.address?.phone}
								<p>
									{data.order?.address?.phone}
								</p>
							{/if}
						</div>

						<div>
							<h5 class="mb-2">Billing Address</h5>

							<p class="flex flex-col">
								<span>
									{data.order?.billingAddress?.firstName}
									{data.order?.billingAddress?.lastName}

									<br />

									{data.order?.billingAddress?.address}, {data.order?.billingAddress?.city},
									{data.order?.billingAddress?.country}, {data.order?.billingAddress?.state}

									<br />

									{data.order?.billingAddress?.zip}
								</span>
							</p>

							{#if data.order?.billingAddress?.phone}
								<p>
									{data.order?.billingAddress?.phone}
								</p>
							{/if}
						</div>

						<!-- <div>
							<h5 class="mb-2">Vendor Details :</h5>

							<p class="flex flex-col">
								<span>
									{data.order?.vendorBusinessName},

									{data.order?.vendorAddress?.address}, {data.order?.vendorAddress?.town},

									{data.order?.vendorAddress?.city}, {data.order?.vendorAddress?.state}</span>

								<span>{data.order?.vendorAddress?.zip}</span>
							</p>

							{#if data.order?.vendorPhone}
								<h6 class="mt-2">
									Phone number: <span> {data.order?.vendorPhone}</span>
								</h6>
							{/if}
						</div> -->
					</div>
				</div>
			</div>

			<!-- Order Tracker -->

			<div>
				{#if !!data.order?.foodType && data.order?.status !== 'delivered' && data.order?.expectedDeliveryDate}
					<div class="mb-5 flex items-center gap-2 flex-wrap">
						<h5>Expected Delivery Date :</h5>

						<p>
							{date(data.order?.expectedDeliveryDate)}
						</p>
					</div>
				{/if}

				<div class="mt-5 sm:mt-10 flex flex-wrap gap-10">
					{#if data.orderTracking?.length}
						<OrderTracking tracks="{data.orderTracking}" />
					{/if}

					<!-- {#if !data.order?.isReplaceOrReturn}
					{:else}
						<ReturnTracking order="{data.order}" />
					{/if} -->

					<div class="flex flex-col gap-2">
						{#if data.order?.invoiceLink}
							<a
								href="{data.order?.invoiceLink}"
								aria-label="Click to download invoice"
								target="blank"
								class="block">
								<PrimaryButton class="w-48" type="button">Download Invoice</PrimaryButton>
							</a>
						{/if}

						{#if data.order?.replaceValidTill != null && now <= data.order?.replaceValidTill && !data.order?.isReplaceOrReturn}
							<a
								href="/my/exchange?orderId=${data.order?.orderId}&itemId=${data.order?.itemId}"
								aria-label="Click to visit exchange"
								class="block">
								<TransparentButton class="w-48" type="button" border>Exchange</TransparentButton>
							</a>
						{/if}

						<!-- {#if data.order?.returnValidTill != null && now <= data.order?.returnValidTill && !data.order?.isReplaceOrReturn}
							<a
								href="/my/return?orderId=${data.order?.orderId}&itemId=${data.order?.itemId}"
								aria-label="Click to visit return"
								class="block">
								<TransparentButton class="w-48" type="button" border>Return</TransparentButton>
							</a>
						{/if} -->
					</div>
				</div>
			</div>
		</section>
	{:else}
		<div class="flex h-[70vh] flex-col items-center justify-center text-center">
			<img src="{noAddToCartAnimate}" alt="empty wishlist" class="mb-5 h-60 object-contain" />

			<h2 class="mb-2">Your have't Ordered Yet !!</h2>

			<p class="mb-5">Add items to it now</p>

			<a href="/" aria-label="Click to visit home" data-sveltekit-preload-data>
				<PrimaryButton class="w-40 py-2 text-sm">Shop Now</PrimaryButton>
			</a>
		</div>
	{/if}
</div>
